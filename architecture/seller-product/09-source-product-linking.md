# Source Product Linking

## How It Works

A seller product can optionally reference a source product from the product-service catalog. This link is established during creation and persists as the `source_product_id` column on `seller_products`.

## What Data Is Pre-Filled From Source

When a seller sources from the catalog, the frontend calls the product-service `GET /find/{productId}` endpoint and receives the full source product data. The following fields are pre-filled into the create form:

| Seller Product Field       | Source Product Field                          | Pre-fill Behavior                           |
| -------------------------- | --------------------------------------------- | ------------------------------------------- |
| title                      | title                                         | Copied directly, seller can edit            |
| description                | description                                   | Copied directly (HTML), seller can edit     |
| properties[].name          | variation.properties[].name                   | Mapped to axes (e.g., "Size", "Color")      |
| properties[].values[].value| variation.properties[].values[].name          | Mapped from property values                 |
| properties[].values[].image_url | variation.properties[].values[].image    | Vendor image URL preserved                  |
| skus[].price               | variation.skus[].price.original               | Converted to local currency as starting point |
| skus[].stock               | stock (product-level)                         | Used as initial stock estimate              |
| skus[].property_values     | variation.skus[].property_associations        | Mapped from property_id + property_value_id |
| specifications[].name      | specifications[].label                        | Copied directly                             |
| specifications[].value     | specifications[].value                        | Copied directly                             |
| gallery[].url              | gallery[].url                                 | Vendor image URLs preserved directly        |

## What the Seller Customizes

After pre-fill, the seller can change anything:

- Adjust pricing to include markup in local currency
- Remove properties they don't want to offer (e.g., only sell Size M and L, not S)
- Add custom specifications not present in the source
- Upload their own images via media center alongside vendor images
- Rewrite the title and description for the local market
- Assign a shipping category (required — the source product doesn't have one)

## The source_product_id Semantics

- **Type:** VARCHAR(30), nullable
- **Value:** The ULID string from product-service (e.g., `01KJJJD0M7E89XNX0EZDNY33V6`)
- **NULL means:** Product was created from scratch (manual creation, Flow B)
- **Non-NULL means:** Product was sourced from vendor catalog (Flow A)
- **No cascading behavior:** If the source product is deleted from product-service, the seller product continues to exist unchanged. The `source_product_id` becomes a stale reference — this is acceptable because the seller product is an independent entity
- **Not unique:** Multiple sellers can source from the same vendor product. The column is indexed but not unique
- **Read-only after creation:** The source link is set during creation and is not changed on update. To re-link, the seller would create a new product
