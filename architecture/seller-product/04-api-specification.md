# API Specification

## Routes (agent-routes.php)

```php
Route::group(['prefix' => 'seller-product', 'middleware' => ['api', 'auth:agent-api', 'verify.agent', 'impersonate']], function () {
    Route::group(['prefix' => 'v1/{agentCompanyId}'], function () {
        Route::group(['prefix' => 'seller-products'], function () {
            Route::get('/', [SellerProductAgentController::class, 'index']);
            Route::get('/status-counts', [SellerProductAgentController::class, 'statusCounts']);
            Route::get('/{sellerProduct}', [SellerProductAgentController::class, 'show']);
            Route::post('/', [SellerProductAgentController::class, 'store']);
            Route::patch('/{sellerProduct}', [SellerProductAgentController::class, 'update']);
            Route::delete('/{sellerProduct}', [SellerProductAgentController::class, 'destroy']);
            Route::post('/bulk-status', [SellerProductAgentController::class, 'bulkStatus']);
            Route::post('/bulk-delete', [SellerProductAgentController::class, 'bulkDelete']);
        });
    });
});
```

**Base URL:** `/api/seller/agent/seller-product/v1/{agentCompanyId}/seller-products`

**Middleware Stack:**
- `api` — rate limiting, CORS
- `auth:agent-api` — JWT authentication for agent users
- `verify.agent` — verify the agent has an active agent company
- `impersonate` — support admin impersonation of agent accounts

**Ownership Resolution:**
The `{agentCompanyId}` path parameter identifies the agent company. The controller verifies that the authenticated user is a member of the specified agent company via `AgentMemberService::isAMember()`. If the user is not a member, the request is rejected with 403 Forbidden.

The owner identity is resolved as a polymorphic pair:
- `owner_type = AgentCompany::class` (`MoveOn\ShippingAgent\Models\AgentCompany`)
- `owner_id = {agentCompanyId}` from the URL path parameter

All queries are scoped by `owner_type` + `owner_id`. No cross-owner access is possible.

**Route Parameters:**
- `{agentCompanyId}` — BIGINT ID of the agent company. The authenticated user must be a member.
- `{sellerProduct}` — BIGINT (auto-increment ID) of the product. Looked up by its integer primary key.

---

## API Workflows

---

### 1. List Products (GET /)

#### What It Accepts

**URL:** `GET /api/seller/agent/seller-product/v1/{agentCompanyId}/seller-products`

**Query Parameters:**

| Parameter              | Required | Description                                                         |
| ---------------------- | -------- | ------------------------------------------------------------------- |
| response_type          | No       | "minimal" or "regular" (default: regular)                           |
| status                 | No       | Filter by status: draft, pending, active, inactive, violation, deleted |
| shipping_category_id   | No       | UUID — filter by shipping category                                  |
| title                  | No       | Partial match search on product title                               |
| sort_by                | No       | Sort field: title, created_at, updated_at, price (default: created_at) |
| sort_order             | No       | asc or desc (default: desc)                                        |
| page                   | No       | Page number (default: 1)                                           |
| per_page               | No       | Items per page, min 1, max 100 (default: 20)                       |
| region_id              | No       | BIGINT — filter products that have at least one active SKU in this region. When provided, the price_range in the list response uses regional prices from `seller_product_sku_regions` instead of base SKU prices. |

#### What It Does

Returns a paginated list of seller products owned by the authenticated agent company.

- If `status` is provided, returns only products with that status
- If `shipping_category_id` is provided, returns only products in that category
- If `title` is provided, performs a case-insensitive partial match search
- If `region_id` is provided, filters to only products that have at least one active SKU in that region (via `seller_product_sku_regions`). When `region_id` is set, the `price_range` in the response uses regional prices from `seller_product_sku_regions` instead of base SKU prices.
- Results always scoped to the authenticated agent company
- Includes the primary gallery image and minimum SKU price in list response for display

#### How It Does It

1. Verify the authenticated user is a member of `{agentCompanyId}` via `AgentMemberService::isAMember()` — if not, return 403. Set `owner_type = AgentCompany::class` and `owner_id = {agentCompanyId}`
2. Validate all query parameters (UUID format, enum values, numeric ranges)
3. Build base query: `WHERE owner_type = {owner_type} AND owner_id = {owner_id}`
4. Apply filters conditionally:
   - `status` → `AND status = {status}`. **Special case:** when `status=deleted` is requested, the query uses `withTrashed()->where('deleted_at', '!=', null)` instead of the standard scope, because `SoftDeletes` normally excludes soft-deleted rows.
   - `shipping_category_id` → `AND shipping_category_id = {category_id}`
   - `title` → `AND title LIKE '%{title}%'`
   - `region_id` → join to `seller_product_sku_regions` via `seller_product_skus`, filter where `seller_product_skus.is_active = true` AND `seller_product_sku_regions.region_id = {region_id}` AND `seller_product_sku_regions.is_active = true`. Both global SKU activation and regional activation must be true (see design decision in database schema). When this filter is active, the price_range is computed from regional `original_price` values instead of base SKU prices.
5. If `sort_by = 'price'`: sort by the denormalized `min_price` column on `seller_products` (no join needed). Note: when `region_id` is provided, sort uses the computed regional min price instead.
6. Otherwise: sort by the specified column directly
7. Eager load: primary gallery image (where `is_primary = true`). Price range source depends on context:
   - **Without `region_id`:** comes from denormalized `min_price` and `max_price` columns on the product row — no SKU join needed.
   - **With `region_id`:** computed as `MIN(original_price)` / `MAX(original_price)` from the `seller_product_sku_regions` rows matched by the join in step 4. The join is already active, so no additional query is needed — the price range is derived from the filtered regional prices.
8. Apply pagination
9. Return paginated collection with `SellerProductAgentCollectionResource`

#### What Is The Outcome

**Success (200):**

```json
{
  "object": "SellerProductCollection",
  "data": [
    {
      "object": "SellerProductMinimal",
      "id": 1,
      "title": "KingBank DDR5 16GB RAM",
      "status": "active",
      "shipping_category_id": "9ce8d7f8-1400-4950-b247-750df997fda3",
      "image": "https://cdn.moveon.global/ingest/images/123-uuid-ts.jpg",
      "price_range": {
        "min": 1200.00,
        "max": 1400.00
      },
      "sku_count": 6,
      "created_at": "2026-02-01T10:00:00.000000Z",
      "updated_at": "2026-02-15T14:30:00.000000Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total": 42,
    "last_page": 3,
    "from": 1,
    "to": 20,
    "path": "/api/seller/agent/seller-product/v1/{agentCompanyId}/seller-products",
    "links": {
      "first": "...?page=1",
      "last": "...?page=3",
      "prev": null,
      "next": "...?page=2"
    }
  }
}
```

**Errors:**

| Status | When                                                          |
| ------ | ------------------------------------------------------------- |
| 403    | User is not a member of the agent company                     |
| 422    | Invalid parameter format (bad UUID, invalid sort field, etc.) |
| 422    | Status value not in SellerProductStatusEnum                   |

---

### 2. Show Product (GET /{id})

#### What It Accepts

**URL:** `GET /api/seller/agent/seller-product/v1/{agentCompanyId}/seller-products/{id}`

**Path Parameters:**

| Parameter      | Required | Description                                |
| -------------- | -------- | ------------------------------------------ |
| agentCompanyId | Yes      | BIGINT ID of the agent company             |
| id             | Yes      | BIGINT ID of the product to retrieve       |

**Query Parameters:**

| Parameter      | Required | Description                                                                     |
| -------------- | -------- | ------------------------------------------------------------------------------- |
| expand[]       | No       | Relations to include: properties, skus, specifications, gallery, category, regional_prices |
| region_id      | No       | BIGINT — when provided, only includes the regional price for the specified region on each SKU (instead of all regions). Requires `expand[]=regional_prices`. |

#### What It Does

Retrieves a single seller product with optional related data.

- By default returns product metadata only
- `expand[]=properties` includes properties with their values
- `expand[]=skus` includes the SKU matrix with property value associations
- `expand[]=specifications` includes specification key-value pairs
- `expand[]=gallery` includes ordered gallery images and videos
- `expand[]=category` includes shipping category name and path
- `expand[]=regional_prices` includes per-region pricing/stock/availability on each SKU. Implicitly includes `expand[]=skus` — if the client sends `expand[]=regional_prices` without `expand[]=skus`, the server treats it as if both were requested, because regional prices are nested inside SKU objects

#### How It Does It

1. Verify the authenticated user is a member of `{agentCompanyId}` via `AgentMemberService::isAMember()` — if not, return 403. Set `owner_type = AgentCompany::class` and `owner_id = {agentCompanyId}`
2. Fetch product by ID where `owner_type = {owner_type} AND owner_id = {owner_id}`. If the product is soft-deleted, use `withTrashed()` so it can still be viewed.
   - If not found → 404 (does not reveal existence to other owners)
3. If `expand[]` includes "properties":
   - Eager load `properties` → ordered by position
   - For each property, load `values` → ordered by position
4. If `expand[]` includes "skus":
   - Eager load `skus` → with `skuAssociations.propertyValue`
5. If `expand[]` includes "specifications":
   - Eager load `specifications` → ordered by position
6. If `expand[]` includes "gallery":
   - Eager load `gallery` → ordered by position
7. If `expand[]` includes "category":
   - Eager load shipping category with parent chain for breadcrumb path
8. If `expand[]` includes "regional_prices":
   - Implicitly include "skus" in the expand set (regional prices are nested inside SKU objects)
   - Eager load `skus.skuRegions`
   - If `region_id` query param is present, filter to only include `skuRegions` where `region_id = {region_id}`
9. Return product with requested relations + `ETag` response header

#### What Is The Outcome

**Success (200):**

Response Headers:
```
ETag: "a1b2c3d4e5f6789012345678abcdef01"
```

```json
{
  "message": "Product retrieved successfully.",
  "data": {
    "object": "SellerProduct",
    "id": 1,
    "title": "KingBank DDR5 16GB RAM",
    "description": "<p>High-performance DDR5 memory...</p>",
    "status": "active",
    "shipping_category_id": "9ce8d7f8-1400-4950-b247-750df997fda3",
    "source_product_id": "01KJJJD0M7E89XNX0EZDNY33V6",
    "created_by": 42,
    "etag": "a1b2c3d4e5f6789012345678abcdef01",
    "lock_version": 3,
    "created_at": "2026-02-01T10:00:00.000000Z",
    "updated_at": "2026-02-15T14:30:00.000000Z",
    "properties": {
      "object": "SellerProductPropertyCollection",
      "data": [
        {
          "object": "SellerProductProperty",
          "id": 1,
          "name": "Size",
          "position": 0,
          "values": {
            "object": "SellerProductPropertyValueCollection",
            "data": [
              {
                "object": "SellerProductPropertyValue",
                "id": 1,
                "value": "DDR5 6000MHZ H",
                "image_url": null,
                "position": 0
              }
            ]
          }
        },
        {
          "object": "SellerProductProperty",
          "id": 2,
          "name": "Color",
          "position": 1,
          "values": {
            "object": "SellerProductPropertyValueCollection",
            "data": [
              {
                "object": "SellerProductPropertyValue",
                "id": 2,
                "value": "silver",
                "image_url": "https://m.media-amazon.com/images/I/41u1jis0GxL.jpg",
                "position": 0
              }
            ]
          }
        }
      ]
    },
    "skus": {
      "object": "SellerProductSkuCollection",
      "data": [
        {
          "object": "SellerProductSku",
          "id": 101,
          "sku_code": null,
          "price": 36573.63,
          "stock": 2,
          "is_active": true,
          "regional_prices": {
            "object": "SellerProductSkuRegionPriceCollection",
            "data": [
              {
                "object": "SellerProductSkuRegionPrice",
                "id": 1,
                "region_id": 1,
                "original_price": 36573.63,
                "discount_price": 34999.00,
                "stock": 50,
                "is_active": true
              },
              {
                "object": "SellerProductSkuRegionPrice",
                "id": 2,
                "region_id": 3,
                "original_price": 449.99,
                "discount_price": null,
                "stock": 20,
                "is_active": true
              }
            ]
          },
          "associations": {
            "object": "SellerProductSkuAssociationCollection",
            "data": [
              {
                "object": "SellerProductSkuAssociation",
                "id": 1,
                "property_id": 1,
                "property_value_id": 1,
                "property": {
                  "id": 1,
                  "name": "Size",
                  "position": 0
                },
                "property_value": {
                  "id": 1,
                  "value": "DDR5 6000MHZ H",
                  "image_url": null,
                  "position": 0
                }
              },
              {
                "object": "SellerProductSkuAssociation",
                "id": 2,
                "property_id": 2,
                "property_value_id": 2,
                "property": {
                  "id": 2,
                  "name": "Color",
                  "position": 1
                },
                "property_value": {
                  "id": 2,
                  "value": "silver",
                  "image_url": "https://m.media-amazon.com/images/I/41u1jis0GxL.jpg",
                  "position": 0
                }
              }
            ]
          }
        }
      ]
    },
    "specifications": {
      "object": "SellerProductSpecificationCollection",
      "data": [
        {
          "object": "SellerProductSpecification",
          "id": 1,
          "name": "Brand",
          "value": "KingBank",
          "type": "text",
          "position": 0
        },
        {
          "object": "SellerProductSpecification",
          "id": 2,
          "name": "RAM Memory Technology",
          "value": "DDR5",
          "type": "text",
          "position": 1
        }
      ]
    },
    "gallery": {
      "object": "SellerProductGalleryCollection",
      "data": [
        {
          "object": "SellerProductGalleryItem",
          "id": 1,
          "asset_id": null,
          "url": "https://m.media-amazon.com/images/I/61jSWw3xpyL._AC_SL1280_.jpg",
          "type": "image",
          "thumbnail_url": null,
          "duration": null,
          "position": 0,
          "is_primary": true
        },
        {
          "object": "SellerProductGalleryItem",
          "id": 2,
          "asset_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
          "url": "https://cdn.moveon.global/ingest/videos/42-uuid-ts.mp4",
          "type": "video",
          "thumbnail_url": "https://cdn.moveon.global/ingest/thumbnails/42-uuid-ts.jpg",
          "duration": 45,
          "position": 1,
          "is_primary": false
        },
        {
          "object": "SellerProductGalleryItem",
          "id": 3,
          "asset_id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
          "url": "https://cdn.moveon.global/ingest/images/42-uuid-ts.jpg",
          "type": "image",
          "thumbnail_url": null,
          "duration": null,
          "position": 2,
          "is_primary": false
        }
      ]
    },
    "category": {
      "object": "ShippingCategory",
      "id": "9ce8d7f8-1400-4950-b247-750df997fda3",
      "name": "Smart Card",
      "path": ["Electronics & Gadgets", "Access Control System", "Smart Card"]
    }
  }
}
```

**Errors:**

| Status | When                                             |
| ------ | ------------------------------------------------ |
| 403    | User is not a member of the agent company        |
| 404    | Product not found or belongs to another owner  |
| 422    | Invalid expand[] value                           |
| 422    | Invalid region_id (not a valid integer)          |

---

### 3. Create Product (POST /)

#### What It Accepts

**URL:** `POST /api/seller/agent/seller-product/v1/{agentCompanyId}/seller-products`

**Request Body:**

| Field                                   | Required | Type            | Description                                              |
| --------------------------------------- | -------- | --------------- | -------------------------------------------------------- |
| shipping_category_id                    | Yes      | uuid            | Leaf category UUID from ShippingCore                     |
| title                                   | Yes      | string          | Product title, 1-255 characters                          |
| description                             | No       | string          | Rich HTML description, max 65535 characters              |
| status                                  | No       | string          | Initial status: draft or pending (default: draft)        |
| source_product_id                       | No       | string          | ULID from product-service (max 30 chars)                 |
| properties                              | No       | array           | Array of properties                                      |
| properties[].name                       | Yes      | string          | Axis name (e.g., "Size"), max 100 chars                  |
| properties[].position                   | Yes      | integer         | Display order (0-indexed)                                |
| properties[].values                     | Yes      | array           | Array of values for this axis                            |
| properties[].values[].value             | Yes      | string          | Value text (e.g., "Red"), max 100 chars                  |
| properties[].values[].image_url         | No       | string          | Swatch image URL, max 500 chars                          |
| properties[].values[].position          | Yes      | integer         | Display order within axis                                |
| skus                                    | No       | array           | Array of SKU definitions                                 |
| skus[].price                            | Yes      | numeric         | Price, min 0.01, max 9999999999.99                       |
| skus[].stock                            | Yes      | integer         | Stock quantity, min 0                                    |
| skus[].sku_code                         | No       | string          | Custom SKU code, max 50 chars                            |
| skus[].is_active                        | No       | boolean         | Default: true                                            |
| skus[].property_values                 | Yes      | array           | Positional index pairs referencing `properties[axisIndex].values[valueIndex]`. During creation, property values don't have IDs yet (they're created in the same request), so SKUs reference values by position: `[axisIndex, valueIndex]` pairs. After creation, the Update endpoint (JSON Patch) uses BIGINT IDs via `property_value_ids` because the values already exist in the database. |
| specifications                          | No       | array           | Array of specification entries                           |
| specifications[].name                   | Yes      | string          | Spec label, max 100 chars                                |
| specifications[].value                  | Yes      | string          | Spec value, max 500 chars                                |
| specifications[].type                   | No       | string          | "text" or "select" (default: text)                       |
| specifications[].position               | Yes      | integer         | Display order                                            |
| gallery                                 | No       | array           | Array of gallery entries (images and videos)             |
| gallery[].asset_id                      | No       | uuid            | Asset UUID from Storage module                           |
| gallery[].url                           | Yes      | string          | Image or video URL, max 500 chars                        |
| gallery[].type                          | No       | string          | "image" or "video" (default: image)                      |
| gallery[].thumbnail_url                 | No       | string          | Video poster/thumbnail URL, max 500 chars. Required when type=video. |
| gallery[].duration                      | No       | integer         | Video duration in seconds, min 0. Optional, for video.   |
| gallery[].position                      | Yes      | integer         | Display order (0 = first)                                |
| gallery[].is_primary                    | No       | boolean         | Default: false. Only applicable when type=image.         |
| skus[].regional_prices                  | No       | array           | Array of per-region pricing objects for this SKU         |
| skus[].regional_prices[].region_id      | Yes      | integer         | Region ID from Core module                               |
| skus[].regional_prices[].original_price | Yes      | numeric         | Full price in region's currency, min 0.01                |
| skus[].regional_prices[].discount_price | No       | numeric/null    | Discounted price, min 0.01. NULL = no discount           |
| skus[].regional_prices[].stock          | Yes      | integer         | Stock in this region, min 0                              |
| skus[].regional_prices[].is_active      | No       | boolean         | Default: true                                            |

#### What It Does

Creates a new seller product with all related data in a single atomic transaction.

- Validates that the shipping category exists and is a leaf node (no children)
- Validates properties and specifications against the category's schema definition
- Validates SKU property_values references are consistent with the submitted properties
- Ensures exactly one gallery image is marked as primary (if gallery is provided). Videos cannot be primary.
- Enforces constraint limits (max 3 properties, max 50 values per property, max 100 SKUs, max 20 gallery items, max 50 specs)
- Validates `sku_code` uniqueness per product when provided

#### How It Does It

1. Verify the authenticated user is a member of `{agentCompanyId}` via `AgentMemberService::isAMember()` — if not, return 403. Set `owner_type = AgentCompany::class`, `owner_id = {agentCompanyId}`, and `created_by = auth()->id()`
2. Validate all request fields (format, types, lengths)
3. Validate `shipping_category_id`:
   - Query ShippingCore to confirm the category exists
   - Confirm the category is a leaf node (has no children)
   - If not found → 404
   - If not a leaf → 422 "Category must be a leaf category"
4. If `properties` provided, validate against category schema:
   - Load the category schema from `moveon-category-schemas.json` mapping
   - Check that required property attributes from the schema are present
   - Check that submitted property names exist in the schema's `variation_attributes`
   - Validate submitted property values against the schema's allowed options
5. If `specifications` provided, validate against category schema:
   - Check that required specification attributes from the schema are present
   - For `type: select` specifications, validate that the submitted value is in the schema's `options` list
6. If `skus` provided:
   - Validate count does not exceed 100
   - Validate each SKU's `property_values` references are consistent — each SKU must reference exactly one value from each property
   - Validate no duplicate SKU combinations
   - If any SKUs have `sku_code` set, validate uniqueness — no two SKUs in the same product can share a `sku_code`
7. If `gallery` provided:
   - Validate count does not exceed 20
   - Validate exactly one image has `is_primary = true`
   - If any `asset_id` provided, verify the asset exists in Storage and belongs to the same owner
8. Begin database transaction
9. Create `seller_products` row with: title, description, status, shipping_category_id, source_product_id, owner_type, owner_id, created_by
10. For each property: create `seller_product_properties` row, then create `seller_product_property_values` rows for its values
11. For each SKU: create `seller_product_skus` row, then create `seller_product_sku_associations` rows — resolve the positional `[axisIndex, valueIndex]` references to the just-created property value IDs, and derive `property_id` from each value's parent property. Each association row stores both `property_id` (the property) and `property_value_id` (the specific value).
12. For each specification: create `seller_product_specifications` row
13. For each gallery item: create `seller_product_gallery` row (with type, thumbnail_url, duration if provided). Validate that `is_primary` is only set on type=image entries, and that type=video entries include a `thumbnail_url`.
14. For each SKU's `regional_prices` (if provided): create `seller_product_sku_regions` rows with region_id, original_price, discount_price, stock, is_active. Validate that each `region_id` references a valid region in the Core module and that no duplicate region per SKU exists.
15. Call `computeAndStore()` — computes initial `etag` from serialized product state (including regional prices) and increments `lock_version` from 0 to 1
16. Commit transaction
17. Return created product with all relations + `ETag` response header

#### What Is The Outcome

**Success (201):**

Response Headers:
```
ETag: "f8c3de3d1c3e..."
```

```json
{
  "message": "Product created successfully.",
  "data": {
    "object": "SellerProduct",
    "id": 42,
    "title": "KingBank DDR5 16GB RAM",
    "status": "draft",
    "shipping_category_id": "9ce8d7f8-1400-4950-b247-750df997fda3",
    "source_product_id": "01KJJJD0M7E89XNX0EZDNY33V6",
    "created_by": 42,
    "etag": "f8c3de3d1c3e...",
    "lock_version": 1,
    "created_at": "2026-03-01T10:00:00.000000Z",
    "updated_at": "2026-03-01T10:00:00.000000Z",
    "properties": { "object": "SellerProductPropertyCollection", "data": ["..."] },
    "skus": { "object": "SellerProductSkuCollection", "data": ["..."] },
    "specifications": { "object": "SellerProductSpecificationCollection", "data": ["..."] },
    "gallery": { "object": "SellerProductGalleryCollection", "data": ["..."] }
  }
}
```

**Errors:**

| Status | When                                                                                   |
| ------ | -------------------------------------------------------------------------------------- |
| 403    | User is not a member of the agent company                                              |
| 404    | shipping_category_id does not exist                                                    |
| 422    | shipping_category_id is not a leaf category                                            |
| 422    | Required property attribute from schema is missing                                     |
| 422    | Submitted property name not in category schema                                         |
| 422    | Submitted property value not in schema's allowed options                               |
| 422    | Required specification attribute from schema is missing                                |
| 422    | Specification value not in schema's allowed options (for select type)                  |
| 422    | SKU count exceeds 100                                                                  |
| 422    | SKU property_values references are inconsistent (missing axis or duplicate axis)       |
| 422    | Duplicate SKU combination (same set of property values)                                |
| 422    | Gallery count exceeds 20                                                               |
| 422    | Gallery has zero or multiple primary images                                            |
| 422    | Gallery video missing thumbnail_url                                                    |
| 422    | is_primary set on a video gallery entry                                                |
| 422    | Validation errors (missing required fields, wrong types, exceeded lengths)             |
| 422    | Regional price region_id does not exist                                                |
| 422    | Duplicate region_id within a SKU's regional_prices                                     |
| 400    | Database transaction failed                                                            |

---

### 4. Update Product (PATCH /{id}) — RFC 6902 JSON Patch + RFC 7232 ETag

#### What It Accepts

**URL:** `PATCH /api/seller/agent/seller-product/v1/{agentCompanyId}/seller-products/{id}`

**Content-Type:** `application/json-patch+json`

**Path Parameters:**

| Parameter      | Required | Description                        |
| -------------- | -------- | ---------------------------------- |
| agentCompanyId | Yes      | BIGINT ID of the agent company     |
| id             | Yes      | BIGINT ID of the product to update |

**Required Headers:**

| Header     | Required | Description                                                        |
| ---------- | -------- | ------------------------------------------------------------------ |
| If-Match   | Yes      | ETag value from the last GET. Format: `"{etag}"` (quoted string). Prevents concurrent modification — if the product changed since the client last fetched it, the server returns 412 and the client must re-fetch. |

**Request Body — RFC 6902 JSON Patch Document:**

The body is a JSON array of patch operations. Each operation targets a specific path in the product resource using JSON Pointer syntax (RFC 6901). The product resource is the same shape returned by the Show endpoint — top-level fields, nested `properties`, `skus`, `specifications`, `gallery` arrays. Nested array elements are addressed by their BIGINT database ID (not array index).

**Supported operations:**

| Operation | RFC 6902 `op` | Description                                                        |
| --------- | ------------- | ------------------------------------------------------------------ |
| Replace   | `replace`     | Change the value at a path. Target must exist.                     |
| Add       | `add`         | Insert a new element at a path. Use `/-` suffix to append.         |
| Remove    | `remove`      | Delete the element at a path. Uses the element's database ID.      |
| Test      | `test`        | Assert a value equals expected. If not, the entire patch fails. Used for safety checks. |

`move` and `copy` are defined in RFC 6902 but not supported by this endpoint. `move` can be expressed as `remove` + `add`. `copy` has no use case for product data — duplicating a SKU or property value is not a meaningful operation.

**Addressable paths:**

| Path Pattern                                  | Op         | Value Type | Description                          |
| --------------------------------------------- | ---------- | ---------- | ------------------------------------ |
| `/title`                                      | replace    | string     | Product title (1-255 chars)          |
| `/description`                                | replace    | string     | Product description (max 65535)      |
| `/status`                                     | replace    | string     | Status transition (validated)        |
| `/shipping_category_id`                       | replace    | string     | Leaf category UUID                   |
| `/properties/-`                               | add        | object     | Append a new property                |
| `/properties/{id}`                            | remove     | -          | Remove property by ID                |
| `/properties/{id}/name`                       | replace    | string     | Rename property                      |
| `/properties/{id}/position`                   | replace    | integer    | Change property display order        |
| `/properties/{id}/values/-`                   | add        | object     | Append a new value to property       |
| `/properties/{id}/values/{id}`                | remove     | -          | Remove a property value by ID        |
| `/properties/{id}/values/{id}/value`          | replace    | string     | Rename a property value              |
| `/properties/{id}/values/{id}/image_url`      | replace    | string     | Change swatch image                  |
| `/properties/{id}/values/{id}/position`       | replace    | integer    | Change value display order           |
| `/skus/-`                                     | add        | object     | Append a new SKU                     |
| `/skus/{id}`                                  | remove     | -          | Remove SKU by ID                     |
| `/skus/{id}/price`                            | replace    | number     | Change SKU price                     |
| `/skus/{id}/stock`                            | replace    | integer    | Change SKU stock                     |
| `/skus/{id}/sku_code`                         | replace    | string     | Change SKU code                      |
| `/skus/{id}/is_active`                        | replace    | boolean    | Toggle SKU availability              |
| `/specifications/-`                           | add        | object     | Append a new specification           |
| `/specifications/{id}`                        | remove     | -          | Remove specification by ID           |
| `/specifications/{id}/name`                   | replace    | string     | Change spec label                    |
| `/specifications/{id}/value`                  | replace    | string     | Change spec value                    |
| `/specifications/{id}/type`                   | replace    | string     | Change spec type (text/select)       |
| `/specifications/{id}/position`               | replace    | integer    | Change spec display order            |
| `/gallery/-`                                  | add        | object     | Append a new gallery item (image or video) |
| `/gallery/{id}`                               | remove     | -          | Remove gallery item by ID            |
| `/gallery/{id}/url`                           | replace    | string     | Change image or video URL            |
| `/gallery/{id}/asset_id`                      | replace    | string     | Change asset reference               |
| `/gallery/{id}/type`                          | replace    | string     | Change media type (image/video)      |
| `/gallery/{id}/thumbnail_url`                 | replace    | string/null | Change video thumbnail URL          |
| `/gallery/{id}/duration`                      | replace    | integer/null | Change video duration in seconds   |
| `/gallery/{id}/position`                      | replace    | integer    | Change display order                 |
| `/gallery/{id}/is_primary`                    | replace    | boolean    | Change primary flag (type=image only) |
| `/skus/{id}/regional_prices/-`                | add        | object     | Add regional price for a SKU         |
| `/skus/{id}/regional_prices/{id}`             | remove     | -          | Remove regional price by ID          |
| `/skus/{id}/regional_prices/{id}/original_price` | replace | number     | Change original price                |
| `/skus/{id}/regional_prices/{id}/discount_price` | replace | number/null | Change discount price               |
| `/skus/{id}/regional_prices/{id}/stock`       | replace    | integer    | Change regional stock                |
| `/skus/{id}/regional_prices/{id}/is_active`   | replace    | boolean    | Toggle regional SKU visibility       |

**Note on `test`:** The `test` operation is valid on all addressable paths listed above, not only those marked with a specific op. Any path that can be read can be tested.

**Why ID-based addressing:** Patch paths use the database BIGINT ID of each nested element (e.g., `/skus/42/price` targets the SKU with `id = 42`). IDs are stable, immutable identifiers — they don't shift when elements are added or removed. The client references specific database records, not ephemeral array positions. ETag (`If-Match`) remains mandatory as a concurrency guard — prevents concurrent modification even though ID-based addressing eliminates index-shifting concerns.

**Important:** Before building a JSON Patch document, the client MUST fetch the product with all relations expanded: `GET /{id}?expand[]=properties&expand[]=skus&expand[]=specifications&expand[]=gallery`. The patch paths reference IDs from these arrays. A GET gives the client the current IDs and structure needed to construct correct paths.

#### What It Does

Applies an ordered sequence of JSON Patch operations to a product and its nested relations in a single atomic transaction. Operations execute left-to-right. If any operation fails (invalid path, type mismatch, failed `test`), the entire patch is rejected and no changes are persisted.

**Key behaviors:**

- **Atomic.** All operations succeed or all fail. No partial application.
- **Ordered.** Operations execute sequentially. Earlier operations affect the state seen by later operations (e.g., after removing a property value, SKU associations referencing it are also removed).
- **ETag-guarded.** The `If-Match` header is mandatory. Requests without it receive `428 Precondition Required`. Requests with a stale etag receive `412 Precondition Failed`.
- **`remove` on a property cascades.** Removing `/properties/{id}` also removes all its values and any SKU associations that reference those values. Orphaned SKUs (with incomplete property associations) are flagged as errors in post-operation validation.
- **`add` with `-` appends.** Using `/properties/-` appends to the end of the array. The `position` field in the added object controls display order.
- **Constraint checks run after all operations.** Total properties (max 3), SKUs (max 100), gallery (max 20), specs (max 50) are counted after the full patch sequence completes.

#### How It Does It

1. Verify the authenticated user is a member of `{agentCompanyId}` via `AgentMemberService::isAMember()` — if not, return 403. Set `owner_type = AgentCompany::class` and `owner_id = {agentCompanyId}`
2. Fetch product by ID where `owner_type = {owner_type} AND owner_id = {owner_id}`
   - If not found → 404 (does not reveal existence to other owners)
3. **ETag check (RFC 7232):**
   - Read `If-Match` header from request
   - If header is missing → 428 Precondition Required
   - Compare header value against product's stored `etag`
   - If mismatch → 412 Precondition Failed (client must re-GET and rebuild patch)
4. Parse and validate the JSON Patch document:
   - Must be a JSON array
   - Each element must have `op` (string) and `path` (string)
   - `add`, `replace`, `test` operations must have `value`
   - `op` must be one of: `add`, `remove`, `replace`, `test`
   - Paths must match addressable paths listed above
5. Load the product's full state as an in-memory document (the same shape as the Show response with all relations expanded)
6. Begin database transaction
7. Apply operations sequentially against the in-memory document:
   - **`test`:** Assert the value at `path` equals `value`. If not → abort with 409 Conflict
   - **`replace`:** Set the value at `path` to `value`. Resolve `{id}` in path to the element with that database ID. If ID not found → abort with 422
   - **`add`:** If path ends with `-`, append the value. The `position` field controls display order.
   - **`remove`:** Delete the element with the specified ID. For properties, cascade-delete child values and SKU associations.
8. After all operations: validate status transition if `/status` was changed:
   - draft → pending, pending → active, active → inactive, inactive → active, violation → pending, any → deleted
   - Invalid transitions → 422
9. After all operations: if `/shipping_category_id` was changed, validate the new category exists and is a leaf, then re-validate all properties and specs against the new category schema
10. Post-operation constraint checks:
    - Count total properties ≤ 3
    - Count total property values per property ≤ 50
    - Count total SKUs ≤ 100
    - Count total specs ≤ 50
    - Count total gallery items (images + videos) ≤ 20
    - If gallery exists, exactly one image must be `is_primary = true`
    - `is_primary` can only be `true` on type=image entries (not on videos)
    - No duplicate property names, no duplicate values within a property, no duplicate spec names
    - No orphaned SKUs (missing property value associations)
    - If category schema exists, all properties and specs conform
11. If any constraint fails → rollback, return 422
12. Persist the in-memory document state to the database (diff against original, issue INSERT/UPDATE/DELETE as needed)
13. Increment `lock_version`, recompute `etag` from new serialized state
14. Commit transaction
15. Return updated product with all relations + new `ETag` response header

#### What Is The Outcome

**Success (200):**

Response Headers:
```
ETag: "a1b2c3d4e5f6..."
```

```json
{
  "message": "Product updated successfully.",
  "data": {
    "object": "SellerProduct",
    "id": 1,
    "title": "Updated Title",
    "status": "active",
    "etag": "a1b2c3d4e5f6...",
    "lock_version": 4,
    "...": "full product with all relations"
  }
}
```

**Example — Change one SKU's price and stock:**

The client GETs the product, sees that the SKU with `id: 101` is the one they want to update, and uses the returned ETag.

```
PATCH /api/seller/agent/seller-product/v1/{agentCompanyId}/seller-products/{id}
Content-Type: application/json-patch+json
If-Match: "a1b2c3d4e5f6..."
```

```json
[
  { "op": "replace", "path": "/skus/101/price", "value": 1500.00 },
  { "op": "replace", "path": "/skus/101/stock", "value": 75 }
]
```

**Example — Add a new Color value and a new SKU, remove a spec:**

```json
[
  { "op": "add", "path": "/properties/2/values/-", "value": { "_tempId": "new-green", "value": "Green", "image_url": null, "position": 2 } },
  { "op": "add", "path": "/skus/-", "value": { "price": 1200.00, "stock": 40, "sku_code": null, "is_active": true, "property_value_ids": [1], "property_value_refs": ["new-green"] } },
  { "op": "remove", "path": "/specifications/3" }
]
```

**Note on adding a property value and an SKU that references it:** When a new property value is added via `add` at `/properties/{id}/values/-`, the server assigns it a BIGINT ID during in-memory application. Subsequent operations in the same patch can reference the new value via the `_tempId` mechanism. Two approaches:

1. **Two-request approach:** First PATCH adds the property value. Client GETs the updated product (with the new value's ID), then sends a second PATCH to add the SKU referencing it.
2. **Single-request with `_tempId`:** The `add` operation for a property value can include a `_tempId` field. SKU `add` operations can reference this `_tempId` in `property_value_refs` instead of IDs. The server resolves `_tempId` → actual ID during in-memory application.

```json
[
  { "op": "add", "path": "/properties/2/values/-", "value": { "_tempId": "new-green", "value": "Green", "image_url": null, "position": 2 } },
  { "op": "add", "path": "/skus/-", "value": { "price": 1200.00, "stock": 40, "is_active": true, "property_value_ids": [1], "property_value_refs": ["new-green"] } }
]
```

This is a pragmatic extension beyond RFC 6902. The `_tempId` field is ignored on the resource itself and only used for intra-patch reference resolution.

**Example — Reorder gallery and change primary image:**

```json
[
  { "op": "replace", "path": "/gallery/1/is_primary", "value": false },
  { "op": "replace", "path": "/gallery/1/position", "value": 2 },
  { "op": "replace", "path": "/gallery/3/is_primary", "value": true },
  { "op": "replace", "path": "/gallery/3/position", "value": 0 }
]
```

**Example — Change title and status only (no nested changes):**

```json
[
  { "op": "replace", "path": "/title", "value": "New Product Name" },
  { "op": "replace", "path": "/status", "value": "active" }
]
```

**Example — Use `test` to assert before modifying (safety check):**

The `test` operation aborts the entire patch if the value doesn't match, preventing blind overwrites:

```json
[
  { "op": "test", "path": "/skus/101/price", "value": 1000.00 },
  { "op": "replace", "path": "/skus/101/price", "value": 1200.00 }
]
```

If SKU 101's price is not `1000.00`, the server returns 409 Conflict and no changes apply.

**Errors:**

| Status | When                                                                                    |
| ------ | --------------------------------------------------------------------------------------- |
| 403    | User is not a member of the agent company                                               |
| 404    | Product not found or belongs to another owner                                         |
| 409    | `test` operation failed — value at path does not match expected                         |
| 412    | `If-Match` etag does not match current product state (concurrent modification)          |
| 415    | Content-Type is not `application/json-patch+json`                                       |
| 422    | Patch document is not a JSON array                                                      |
| 422    | Operation missing required field (`op`, `path`, or `value`)                             |
| 422    | `op` not one of: add, remove, replace, test                                             |
| 422    | Path does not match any addressable path                                                |
| 422    | `replace` on a path that does not exist                                                 |
| 422    | ID not found in the collection                                                          |
| 422    | Invalid status transition                                                               |
| 422    | `add` on SKU but `property_value_ids` reference non-existent values                    |
| 422    | `_tempId` referenced in `property_value_refs` but no `add` operation defines it        |
| 422    | Post-operation: total properties exceed 3                                                |
| 422    | Post-operation: total SKUs exceed 100                                                    |
| 422    | Post-operation: total specs exceed 50                                                    |
| 422    | Post-operation: total gallery exceed 20                                                  |
| 422    | Post-operation: zero or multiple primary gallery images                                  |
| 422    | `replace` type to video on primary gallery entry                                         |
| 422    | `replace` type to video without thumbnail_url                                            |
| 422    | `add` gallery video without thumbnail_url                                                |
| 422    | `add` gallery video with is_primary=true                                                 |
| 422    | Post-operation: is_primary set on a video gallery entry                                  |
| 422    | Post-operation: duplicate property names                                                 |
| 422    | Post-operation: duplicate spec names                                                     |
| 422    | Post-operation: orphaned SKU (missing property value association after value removal)     |
| 422    | Post-operation: properties or specs violate category schema (after category change)       |
| 422    | `add` on regional price but `region_id` does not exist                                   |
| 422    | `add` on regional price but duplicate `region_id` for this SKU                           |
| 428    | `If-Match` header missing (precondition required)                                       |
| 400    | Database transaction failure                                                             |

---

### 5. Delete Product (DELETE /{id})

#### What It Accepts

**URL:** `DELETE /api/seller/agent/seller-product/v1/{agentCompanyId}/seller-products/{id}`

**Path Parameters:**

| Parameter      | Required | Description                        |
| -------------- | -------- | ---------------------------------- |
| agentCompanyId | Yes      | BIGINT ID of the agent company     |
| id             | Yes      | BIGINT ID of the product to delete |

#### What It Does

Soft deletes a product by setting its status to `deleted` and populating `deleted_at`. All related data (properties, SKUs, specs, gallery) remains in the database but the product no longer appears in listings unless explicitly filtered by `status=deleted`.

#### How It Does It

1. Verify the authenticated user is a member of `{agentCompanyId}` via `AgentMemberService::isAMember()` — if not, return 403. Set `owner_type = AgentCompany::class` and `owner_id = {agentCompanyId}`
2. Fetch product by ID where `owner_type = {owner_type} AND owner_id = {owner_id}`
   - If not found → 404
3. If product is already deleted → return 200 (idempotent)
4. Update product: `status = 'deleted'`
5. Soft delete the product (sets `deleted_at` timestamp)
6. Return success

#### What Is The Outcome

**Success (200):**

```json
{
  "message": "Product deleted successfully."
}
```

**Errors:**

| Status | When                                            |
| ------ | ----------------------------------------------- |
| 403    | User is not a member of the agent company       |
| 404    | Product not found or belongs to another owner |

---

### 6. Bulk Status Update (POST /bulk-status)

#### What It Accepts

**URL:** `POST /api/seller/agent/seller-product/v1/{agentCompanyId}/seller-products/bulk-status`

**Request Body:**

| Field       | Required | Type       | Description                                  |
| ----------- | -------- | ---------- | -------------------------------------------- |
| product_ids | Yes      | integer[]  | Array of product BIGINT IDs (min 1, max 50)  |
| status      | Yes      | string     | Target status: active, inactive, deleted     |

#### What It Does

Changes the status of multiple products in a single operation. Only allowed target statuses for bulk operations are: `active`, `inactive`, `deleted`. This prevents accidental bulk transitions to `draft`, `pending`, or `violation` (which are per-product lifecycle events, not bulk actions).

#### How It Does It

1. Verify the authenticated user is a member of `{agentCompanyId}` via `AgentMemberService::isAMember()` — if not, return 403. Set `owner_type = AgentCompany::class` and `owner_id = {agentCompanyId}`
2. Validate `status` is one of: active, inactive, deleted
3. Fetch all products where `id IN (product_ids) AND owner_type = {owner_type} AND owner_id = {owner_id}`
4. Count mismatch check:
   - If fetched count < submitted count → 403 "Some products not found or unauthorized"
5. For each product, validate the status transition is allowed:
   - Skip products that are already at the target status
   - Collect products with invalid transitions (e.g., draft → active is not valid) into the `skipped` array with a reason
6. Products with invalid transitions are not updated — they are included in the `skipped` array in the 200 response (not a 422)
7. Begin transaction
8. Update all valid products to the target status
9. If target is `deleted`, soft delete them
10. Commit transaction
11. Return updated count and any skipped/failed details

#### What Is The Outcome

**Success (200):**

```json
{
  "message": "Bulk status update completed.",
  "updated_count": 8,
  "skipped_count": 2,
  "skipped": [
    {
      "id": 15,
      "reason": "Already at target status"
    },
    {
      "id": 23,
      "reason": "Invalid transition from draft to active"
    }
  ]
}
```

**Errors:**

| Status | When                                               |
| ------ | -------------------------------------------------- |
| 403    | User is not a member of the agent company          |
| 403    | Some product_ids belong to another owner         |
| 422    | Empty product_ids array                            |
| 422    | product_ids exceeds 50 items                       |
| 422    | target status not in allowed bulk statuses         |
| 422    | Invalid ID in product_ids                          |

---

### 7. Bulk Delete (POST /bulk-delete)

#### What It Accepts

**URL:** `POST /api/seller/agent/seller-product/v1/{agentCompanyId}/seller-products/bulk-delete`

**Request Body:**

| Field       | Required | Type       | Description                              |
| ----------- | -------- | ---------- | ---------------------------------------- |
| product_ids | Yes      | integer[]  | Array of product BIGINT IDs (min 1, max 50) |

#### What It Does

Soft deletes multiple products in a single operation. This is a convenience endpoint equivalent to `bulk-status` with `status=deleted` but with simpler request body.

#### How It Does It

1. Verify the authenticated user is a member of `{agentCompanyId}` via `AgentMemberService::isAMember()` — if not, return 403. Set `owner_type = AgentCompany::class` and `owner_id = {agentCompanyId}`
2. Fetch all products where `id IN (product_ids) AND owner_type = {owner_type} AND owner_id = {owner_id}`
3. Count mismatch check → 403 if any are missing or unauthorized
4. Begin transaction
5. Update all products: `status = 'deleted'`, soft delete (set `deleted_at`)
6. Skip already-deleted products (idempotent)
7. Commit transaction
8. Return deleted count

#### What Is The Outcome

**Success (200):**

```json
{
  "message": "Products deleted successfully.",
  "deleted_count": 5
}
```

**Errors:**

| Status | When                                               |
| ------ | -------------------------------------------------- |
| 403    | User is not a member of the agent company          |
| 403    | Some product_ids belong to another owner         |
| 422    | Empty product_ids array                            |
| 422    | product_ids exceeds 50 items                       |
| 422    | Invalid ID in product_ids                          |

---

### 8. Product Status Counts (GET /status-counts)

#### What It Accepts

**URL:** `GET /api/seller/agent/seller-product/v1/{agentCompanyId}/seller-products/status-counts`

No query parameters.

#### What It Does

Returns the count of products for each status value. Used to populate tab badges in the product list UI (e.g., "Active (42)", "Draft (7)").

#### How It Does It

1. Verify the authenticated user is a member of `{agentCompanyId}` via `AgentMemberService::isAMember()` — if not, return 403. Set `owner_type = AgentCompany::class` and `owner_id = {agentCompanyId}`
2. Execute a single query:
   ```sql
   SELECT status, COUNT(*) as count
   FROM seller_products
   WHERE owner_type = {owner_type}
     AND owner_id = {owner_id}
     AND deleted_at IS NULL
   GROUP BY status
   ```
3. Include the `deleted` status by running a separate count on soft-deleted rows using `withTrashed()->where('deleted_at', '!=', null)`:
   ```sql
   SELECT COUNT(*) as count
   FROM seller_products
   WHERE owner_type = {owner_type}
     AND owner_id = {owner_id}
     AND deleted_at IS NOT NULL
   ```
4. Merge results and fill missing statuses with 0
5. Compute total (`all` count = sum of all statuses including deleted)

#### What Is The Outcome

**Success (200):**

```json
{
  "object": "SellerProductStatusCounts",
  "data": {
    "all": 67,
    "draft": 7,
    "pending": 3,
    "active": 42,
    "inactive": 10,
    "violation": 2,
    "deleted": 3
  }
}
```

**Errors:**

| Status | When                                            |
| ------ | ----------------------------------------------- |
| 403    | User is not a member of the agent company       |
