# Implementation Notes

## Repository Pattern

- `SellerProductRepository` extends `BaseRepository` from Core module
- Standard methods: `getAll()`, `create()`, `update()`, `find()`, `exists()`
- Uses `ArrayHelper::getFiltersValues()` for filter extraction
- Enum-based field and filter references for type safety

## Transaction Handling

Create and Update operations wrap all database writes in a transaction:

```php
DB::beginTransaction();
try {
    $product = $this->productRepository->create($payload);
    $this->propertyService->syncProperties($product, $propertiesData);
    $this->skuService->syncSkus($product, $skusData);
    $this->skuRegionPriceService->syncRegionalPrices($product, $regionalPricesData);
    // ... specs, gallery
    $this->etagService->computeAndStore($product);
    DB::commit();
} catch (Exception $exception) {
    DB::rollBack();
    throw new SellerProductCreateException($exception);
}
```

## ETag and Optimistic Concurrency (RFC 7232)

The `SellerProductEtagService` handles etag computation and validation:

```php
class SellerProductEtagService
{
    public function computeAndStore(SellerProduct $product): void
    {
        $product->load([
            'properties.values',
            'skus.skuAssociations',
            'skus.skuRegions',
            'specifications',
            'gallery',
        ]);
        $serialized = json_encode($product->toArray());
        $product->etag = md5($serialized);
        $product->lock_version = $product->lock_version + 1;
        $product->save(); // uses saveQuietly internally to avoid observer loops
    }

    public function validateIfMatch(SellerProduct $product, string $ifMatch): void
    {
        $clientEtag = trim($ifMatch, '"');
        if ($clientEtag !== $product->etag) {
            throw new EtagMismatchException();
        }
    }
}
```

The Show and Create endpoints return the `ETag` as both a response header (`ETag: "..."`) and a field in the JSON body (`etag`). The response header follows RFC 7232 convention. The body field is a convenience for frontend clients that prefer reading JSON over parsing headers.

## SKU Association Resolution

When creating or updating SKU associations, the service layer populates both `property_id` and `property_value_id` on each `seller_product_sku_associations` row:

**During Create (POST):**
The client submits SKUs with positional `property_values` references (`[axisIndex, valueIndex]` pairs) because the property values don't have database IDs yet. The service layer:

1. Creates all properties and their values first, capturing the generated BIGINT IDs
2. Builds a lookup map: `[axisIndex][valueIndex] → { property_id, property_value_id }`
3. For each SKU's `property_values` array, resolves the positional pairs to actual IDs
4. Creates `seller_product_sku_associations` rows with both `property_id` (from the property) and `property_value_id` (from the specific value)

**During Update (PATCH) — adding a new SKU:**
The client submits `property_value_ids` (BIGINT IDs of existing values). The service layer:

1. For each `property_value_id`, queries the `seller_product_property_values` table to get the parent `property_id`
2. Creates `seller_product_sku_associations` rows with both `property_id` and `property_value_id`

The client never provides `property_id` directly — it is always derived from the property value's parent.

## JSON Patch Processing (RFC 6902)

The `JsonPatchProcessor` service applies a JSON Patch document against a product's in-memory state:

1. Loads the product with all relations into an array structure matching the Show response shape
2. Iterates through the operations array in order
3. For each operation, resolves the JSON Pointer path — for nested collections (properties, skus, specifications, gallery), the path segment after the collection name is the BIGINT database ID of the target element, not an array index
4. Applies the operation (add/remove/replace/test) against the in-memory array
5. After all operations complete, diffs the resulting array against the original to determine which database rows need INSERT, UPDATE, or DELETE
6. Returns the diff as a set of database operations for the service layer to execute within a transaction

Paths are validated against the whitelist of addressable paths. Any path not in the whitelist is rejected before the operation executes.

### `_tempId` Intra-Patch Reference Resolution

When a single PATCH request needs to add a new property value AND a new SKU that references it, the value doesn't have a database ID yet. The `_tempId` mechanism solves this:

1. The client includes `"_tempId": "some-ref"` in the `add` operation for a property value
2. The processor maintains a map: `tempId → generated BIGINT ID`
3. When the processor encounters an `add` for a SKU with `property_value_refs`, it resolves each ref against the tempId map
4. The resolved IDs are merged into the SKU's `property_value_ids` before the association rows are created
5. `_tempId` is stripped from the persisted data — it is never stored in the database or returned in responses

```php
// Simplified flow inside JsonPatchProcessor
$tempIdMap = [];

// When processing: add /properties/{id}/values/-
$newValue = $this->propertyValueRepo->create($valueData);
if (isset($valueData['_tempId'])) {
    $tempIdMap[$valueData['_tempId']] = $newValue->id;
}

// When processing: add /skus/-
$resolvedValueIds = $skuData['property_value_ids'] ?? [];
foreach ($skuData['property_value_refs'] ?? [] as $ref) {
    if (!isset($tempIdMap[$ref])) {
        throw new JsonPatchException("The temporary reference '{$ref}' was not found.");
    }
    $resolvedValueIds[] = $tempIdMap[$ref];
}
// Create SKU with $resolvedValueIds
```

## Model Traits

All models in the Seller module use:

```php
use HasFactory, SoftDeletes; // SoftDeletes only on SellerProduct
```

Auto-increment BIGINT primary keys are the Laravel default — no UUID trait needed. Child models (properties, SKUs, specs, gallery) do not use `SoftDeletes` — they cascade delete with their parent product.

## Ownership Scoping

Every query is scoped by `owner_type` + `owner_id`. The authentication middleware resolves the polymorphic owner from the request context:
- **Agent API** (`auth:agent-api`): `owner_type = 'MoveOn\ShippingAgent\Models\AgentCompany'`, `owner_id = agent_company_id`
- **User API** (`auth:user-api`): `owner_type = 'MoveOn\Core\Models\User'`, `owner_id = user_id`

The resolved pair is passed to the service layer and forwarded as filters to the repository. No controller method operates without this scope.

## Message Broker Events

Product lifecycle changes dispatch events for downstream consumers:

| Event                        | When                                           |
| ---------------------------- | ---------------------------------------------- |
| SELLER_PRODUCT_CREATE        | Product created                                |
| SELLER_PRODUCT_UPDATE        | Product updated                                |
| SELLER_PRODUCT_STATUS_CHANGE | Status transitions (activate, deactivate, etc.) |
| SELLER_PRODUCT_DELETE        | Product soft deleted                           |

## Regional Pricing Service

The `SellerProductSkuRegionPriceService` manages per-region pricing, stock, and availability for SKUs.

**During Create (POST):**
For each SKU that includes `regional_prices`, the service creates `seller_product_sku_regions` rows after the SKU itself is created. It validates:
- Each `region_id` exists in the Core `regions` table
- No duplicate `region_id` within the same SKU
- `original_price` is positive, `discount_price` (if set) is positive and does not exceed `original_price`

**During Update (PATCH):**
The JSON Patch processor supports:
- `add /skus/{id}/regional_prices/-` — creates a new regional price for a SKU
- `remove /skus/{id}/regional_prices/{id}` — deletes a regional price
- `replace /skus/{id}/regional_prices/{id}/{field}` — updates individual fields

When a SKU is removed via `remove /skus/{id}`, all its regional prices cascade-delete via the FK constraint.

**Currency derivation:**
Currency is NOT stored on `seller_product_sku_regions`. It is derived from `region.currency_id` at query time. This avoids data duplication and stays consistent with the `AddonServicePrice` pattern.

**Activation logic:**
A SKU is purchasable in a region only when:
1. `seller_product_skus.is_active = true` (global toggle), AND
2. `seller_product_sku_regions.is_active = true` (regional toggle)

