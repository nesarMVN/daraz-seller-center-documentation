# Database Schema

## seller_products

The main product record. One row per seller product.

| Column                 | Type          | Nullable | Default | Description                                              |
| ---------------------- | ------------- | -------- | ------- | -------------------------------------------------------- |
| id                     | BIGINT UNSIGNED | No     | AUTO_INCREMENT | Primary key                                       |
| owner_type             | VARCHAR(255)    | No     |         | Polymorphic owner class (e.g., `MoveOn\ShippingAgent\Models\AgentCompany`, `MoveOn\Core\Models\User`) |
| owner_id               | BIGINT UNSIGNED | No     |         | ID of the owning entity (polymorphic — references `agent_companies.id` or `users.id` depending on `owner_type`) |
| shipping_category_id   | UUID          | No       |         | FK to shipping_categories.id — leaf category only        |
| source_product_id      | VARCHAR(30)   | Yes      | NULL    | ULID from product-service. NULL = manual creation        |
| title                  | VARCHAR(255)  | No       |         | Product title (seller-defined)                           |
| description            | TEXT          | Yes      | NULL    | Rich HTML description                                    |
| status                 | VARCHAR(20)   | No       | 'draft' | Enum: draft, pending, active, inactive, violation, deleted |
| created_by             | BIGINT UNSIGNED | No     |         | User ID who created the product                          |
| etag                   | VARCHAR(64)   | No       | ''      | MD5 hash of serialized product state (RFC 7232). Defaults to empty string; `computeAndStore()` overwrites it within the same transaction after all relations are created. |
| lock_version           | BIGINT UNSIGNED | No     | 0       | Monotonic counter, incremented on every write via `computeAndStore()` |
| min_price              | DECIMAL(12,2) | Yes      | NULL    | Denormalized MIN(skus.price). Updated by service layer when SKUs change. |
| max_price              | DECIMAL(12,2) | Yes      | NULL    | Denormalized MAX(skus.price). Updated by service layer when SKUs change. |
| created_at             | TIMESTAMP     | No       |         | Laravel managed                                          |
| updated_at             | TIMESTAMP     | No       |         | Laravel managed                                          |
| deleted_at             | TIMESTAMP     | Yes      | NULL    | Soft delete timestamp                                    |

**Indexes:**

- `PRIMARY KEY (id)`
- `INDEX (owner_type, owner_id, status)` — list products by status per owner
- `INDEX (owner_type, owner_id, shipping_category_id)` — filter by category per owner
- `INDEX (shipping_category_id)` — join to shipping categories
- `INDEX (source_product_id)` — lookup by source product
- `INDEX (status, owner_type, owner_id, created_at)` — status tab queries with sort
- `INDEX (owner_type, owner_id, min_price)` — sort by price queries
- `INDEX (title)` — supports title search queries

**Foreign Keys:**

- `shipping_category_id REFERENCES shipping_categories(id)`

**Design Decision — Why `LIKE` instead of `FULLTEXT` for title search:**
The title search uses `LIKE '%{term}%'` for substring matching (e.g., searching "Bank" finds "KingBank DDR5"). `FULLTEXT` with `MATCH...AGAINST` only supports word-level and prefix matching, not arbitrary substring matching. Since queries are always scoped by owner (via the compound `(owner_type, owner_id, ...)` indexes), the scan is limited to one owner's products — typically hundreds to low thousands of rows — where `LIKE` performance is acceptable. If search volume or dataset size grows significantly, a dedicated search engine (Elasticsearch) would replace this entirely rather than switching to `FULLTEXT`.

**Design Decision — Why polymorphic `owner_type` + `owner_id` instead of a direct FK:**
Products need to support multiple owner types: `AgentCompany` (managed sellers acting through an agent company) and `User` (direct/independent sellers). A direct FK like `agent_company_id` can only reference one table. Polymorphic ownership (`owner_type` + `owner_id`) lets the same `seller_products` table serve both ownership models without schema changes. This follows the existing `owner()` pattern used by `MetaField`, `ShipmentProductImage`, and `ShippingDuration` models in moveon-red. The `created_by` column continues to track which specific user created the product.

**Design Decision — Why no FK constraint on `owner_type` + `owner_id`:**
MySQL does not support foreign key constraints on polymorphic columns because the target table varies by `owner_type`. Referential integrity is enforced at the application layer — the middleware resolves the authenticated owner before any query, and the service layer validates that the owner exists before creating a product.

**Design Decision — Why no `Relation::morphMap()` registration:**
The codebase stores fully qualified class names in `owner_type` (e.g., `MoveOn\ShippingAgent\Models\AgentCompany`). This is the existing convention used by other polymorphic relationships in moveon-red. No `morphMap` alias is registered — the class name is the canonical identifier.

**Design Decision — Why `owner_type` + `owner_id` naming convention:**
The `owner_type` + `owner_id` column names match the existing polymorphic pattern used by `MetaField`, `ShipmentProductImage`, and `ShippingDuration` in moveon-red. Consistency with the established codebase convention avoids confusion and allows reuse of shared scopes and traits.

**Design Decision — Why `source_product_id` is VARCHAR(30) and not a UUID FK:**
The source product lives in a separate service (product-service) with ULID primary keys (26-char string like `01KJJJD0M7E89XNX0EZDNY33V6`). There is no foreign key constraint because the product-service database is not the same database. This is a soft reference — the ID is stored for traceability and pre-fill, but the seller product does not depend on the source product existing.

**Design Decision — `etag` and `lock_version` for optimistic concurrency (RFC 7232):**
The Update endpoint uses RFC 6902 JSON Patch, which addresses nested elements by ID (e.g., `/skus/42/price`). The `etag` + `lock_version` pattern guards against concurrent modification: the client must send `If-Match: "{etag}"` with every PATCH. If the etag doesn't match the current state, the server returns `412 Precondition Failed` and the client must re-fetch before retrying. This pattern already exists in the product-service (`InternalProductVersion` model) and Store module — we follow the same approach. The `etag` is an MD5 hash of the product's serialized state (all relations included). The `lock_version` is a monotonic counter that serves two purposes: (1) audit trail — shows how many times a product has been modified, and (2) lightweight etag regeneration — when the server needs to check if the product changed, it can compare lock_version instead of recomputing the full MD5 hash.

**Design Decision — Dual deletion tracking (`status = 'deleted'` + `deleted_at`):**
`deleted_at` is the source of truth for soft deletion. The `status = 'deleted'` value is maintained in sync for query convenience (allows `GROUP BY status` to include deleted count). The service layer always sets both together — never one without the other.

**Design Decision — Denormalized `min_price` and `max_price`:**
These columns cache the minimum and maximum SKU prices on the product row. The service layer updates them whenever SKUs are created, updated, or deleted (within the same transaction). This avoids a join to `seller_product_skus` for list queries sorted by price, and enables the `(owner_type, owner_id, min_price)` index to handle `sort_by=price` efficiently. The list endpoint reads these columns directly instead of computing `MIN(price)` via a subquery.

## seller_product_properties

Properties (variation axes) for a product. Each row is one property axis (e.g., "Size", "Color").

| Column      | Type         | Nullable | Default           | Description                          |
| ----------- | ------------ | -------- | ----------------- | ------------------------------------ |
| id          | BIGINT UNSIGNED | No    | AUTO_INCREMENT    | Primary key                          |
| product_id  | BIGINT UNSIGNED | No    |                   | FK to seller_products.id             |
| name        | VARCHAR(100) | No       |                   | Property name (e.g., "Size")   |
| position    | SMALLINT UNSIGNED | No  | 0                 | Display order (0-indexed)            |
| created_at  | TIMESTAMP    | No       |                   |                                      |
| updated_at  | TIMESTAMP    | No       |                   |                                      |

**Indexes:**

- `PRIMARY KEY (id)`
- `INDEX (product_id, position)` — ordered list per product
- `UNIQUE (product_id, name)` — no duplicate axis names per product

**Foreign Keys:**

- `product_id REFERENCES seller_products(id) ON DELETE CASCADE`

**Design Decision — Position field behavior:**
Position values are non-negative integers. Gaps are allowed (0, 2, 5 is valid). The server does not normalize or resequence positions — it stores them as provided and returns them sorted ascending. Duplicate positions within the same parent are allowed but discouraged; the sort order of items with equal positions is undefined. This applies to all tables that use a `position` column: `seller_product_properties`, `seller_product_property_values`, `seller_product_skus` (n/a), `seller_product_specifications`, and `seller_product_gallery`.

**Design Decision — Why a separate properties table instead of JSON:**
The source product (product-service) stores properties as nested JSON inside a single `product` JSON column. That works for a read-only catalog where the shape is fixed. For seller products, properties are editable entities — sellers add/remove axes, reorder them, and the values under each axis link to SKUs via the association table. A normalized schema enables direct querying (e.g., "find all products with a Color property"), validation at the DB level (unique constraint on name per product), and clean cascade deletes.

## seller_product_property_values

Individual values under a property axis. Each row is one option (e.g., "Red" under "Color").

| Column        | Type         | Nullable | Default           | Description                                  |
| ------------- | ------------ | -------- | ----------------- | -------------------------------------------- |
| id            | BIGINT UNSIGNED | No    | AUTO_INCREMENT    | Primary key                                  |
| property_id  | BIGINT UNSIGNED | No    |                   | FK to seller_product_properties.id           |
| value         | VARCHAR(100) | No       |                   | Display value (e.g., "Red", "XL")            |
| image_url     | VARCHAR(500) | Yes      | NULL              | Optional swatch image URL                    |
| position      | SMALLINT UNSIGNED | No  | 0                 | Display order within the axis                |
| created_at    | TIMESTAMP    | No       |                   |                                              |
| updated_at    | TIMESTAMP    | No       |                   |                                              |

**Indexes:**

- `PRIMARY KEY (id)`
- `INDEX (property_id, position)` — ordered list per axis
- `UNIQUE (property_id, value)` — no duplicate values per axis

**Foreign Keys:**

- `property_id REFERENCES seller_product_properties(id) ON DELETE CASCADE`

**Design Decision — Why `image_url` is a URL string and not an asset FK:**
The source product data from product-service stores property value images as full URLs (e.g., `https://m.media-amazon.com/images/I/41u1jis0GxL.jpg`). When a seller sources from catalog, these URLs are preserved directly. When uploading custom swatch images, the frontend resolves the asset to a CDN URL before submitting. Storing a URL string keeps the column uniform regardless of image origin.

## seller_product_skus

Individual SKUs representing a specific combination of property values.

| Column      | Type             | Nullable | Default           | Description                                    |
| ----------- | ---------------- | -------- | ----------------- | ---------------------------------------------- |
| id          | BIGINT UNSIGNED  | No       | AUTO_INCREMENT    | Primary key                                    |
| product_id  | BIGINT UNSIGNED  | No       |                   | FK to seller_products.id                       |
| sku_code    | VARCHAR(50)      | Yes      | NULL              | Seller-defined SKU code (optional)             |
| price       | DECIMAL(12,2)    | No       |                   | Selling price in local currency (BDT)          |
| stock       | INT UNSIGNED     | No       | 0                 | Available stock quantity                       |
| is_active   | BOOLEAN          | No       | true              | Whether this SKU is available for purchase     |
| created_at  | TIMESTAMP        | No       |                   |                                                |
| updated_at  | TIMESTAMP        | No       |                   |                                                |

**Indexes:**

- `PRIMARY KEY (id)`
- `INDEX (product_id, is_active)` — active SKUs per product
- `INDEX (product_id)` — all SKUs for a product
- `UNIQUE (product_id, sku_code) WHERE sku_code IS NOT NULL` — partial unique index ensuring sku_code is unique per product when provided

**Foreign Keys:**

- `product_id REFERENCES seller_products(id) ON DELETE CASCADE`

**Design Decision — Why price is on SKU and not on product:**
Different property combinations have different prices. A "Size L" t-shirt costs more than "Size S". The source product data from product-service also stores price per SKU (`variation.skus[].price`). Placing price on the SKU row matches the source data shape and the business reality that price varies by combination.

**Design Decision — Why no separate wholesale pricing table yet:**
The source product data has a `wholesales` field on each SKU price, but it's always empty in current data. Wholesale tiers can be added later as a `seller_product_sku_wholesale_prices` table without modifying the existing schema.

## seller_product_sku_associations

Join table linking each SKU to its specific property values. A SKU with 2 properties (Size + Color) has 2 rows in this table. Each row records both the property and the specific value (property value), mirroring the source product's `property_associations` structure.

| Column              | Type            | Nullable | Default        | Description                                       |
| ------------------- | --------------- | -------- | -------------- | ------------------------------------------------- |
| id                  | BIGINT UNSIGNED | No       | AUTO_INCREMENT | Primary key                                       |
| sku_id              | BIGINT UNSIGNED | No       |                | FK to seller_product_skus.id                      |
| property_id        | BIGINT UNSIGNED | No       |                | FK to seller_product_properties.id — the property axis |
| property_value_id  | BIGINT UNSIGNED | No       |                | FK to seller_product_property_values.id — the specific value (property value) |

**Indexes:**

- `PRIMARY KEY (id)`
- `UNIQUE (sku_id, property_value_id)` — prevent duplicate associations
- `INDEX (property_value_id)` — reverse lookup by value
- `INDEX (property_id)` — reverse lookup by property axis

**Foreign Keys:**

- `sku_id REFERENCES seller_product_skus(id) ON DELETE CASCADE`
- `property_id REFERENCES seller_product_properties(id) ON DELETE CASCADE`
- `property_value_id REFERENCES seller_product_property_values(id) ON DELETE CASCADE`

**Design Decision — Why a join table instead of a JSON column on SKU:**
The source product uses `property_associations` as a nested array on each SKU. For the seller module, a join table enables: (1) DB-level referential integrity between SKU and property value, (2) querying SKUs by property value (e.g., "all SKUs with Color = Red"), (3) cascade deletes when a property value is removed — all associated SKUs lose that link cleanly.

**Design Decision — Why `property_id` is stored alongside `property_value_id`:**
The `property_id` is technically derivable from `property_value_id` via the `seller_product_property_values.property_id` FK. However, storing it directly on the association row provides: (1) query efficiency — no join through `property_values` to determine which axis a SKU belongs to, (2) a direct constraint ensuring the association references a valid axis, (3) alignment with the source product's `property_associations` structure which carries both `property_id` and `property_value_id`. The service layer sets `property_id` from `property_value.property_id` during creation — the client never provides it directly.

## seller_product_specifications

Product specification key-value pairs. Each row is one specification field (e.g., "Brand": "KingBank").

| Column      | Type         | Nullable | Default           | Description                                        |
| ----------- | ------------ | -------- | ----------------- | -------------------------------------------------- |
| id          | BIGINT UNSIGNED | No    | AUTO_INCREMENT    | Primary key                                        |
| product_id  | BIGINT UNSIGNED | No    |                   | FK to seller_products.id                           |
| name        | VARCHAR(100) | No       |                   | Specification label (e.g., "Brand", "Voltage")     |
| value       | VARCHAR(500) | No       |                   | Specification value (e.g., "KingBank", "1.35V")    |
| type        | VARCHAR(20)  | No       | 'text'            | Field type from schema: text, select               |
| position    | SMALLINT UNSIGNED | No  | 0                 | Display order                                      |
| created_at  | TIMESTAMP    | No       |                   |                                                    |
| updated_at  | TIMESTAMP    | No       |                   |                                                    |

**Indexes:**

- `PRIMARY KEY (id)`
- `INDEX (product_id, position)` — ordered list per product
- `UNIQUE (product_id, name)` — no duplicate spec names per product

**Foreign Keys:**

- `product_id REFERENCES seller_products(id) ON DELETE CASCADE`

**Design Decision — Why flat key-value rows instead of JSON:**
The source product stores specifications as `[{label, value}]` arrays. A flat table allows: (1) indexing and querying by specification name across products, (2) validation of uniqueness at DB level, (3) individual spec updates without rewriting the full JSON blob.

## seller_product_gallery

Product images and videos with ordering. Each row links a product to an asset in the Storage module.

| Column      | Type             | Nullable | Default           | Description                                       |
| ----------- | ---------------- | -------- | ----------------- | ------------------------------------------------- |
| id          | BIGINT UNSIGNED  | No       | AUTO_INCREMENT    | Primary key                                       |
| product_id  | BIGINT UNSIGNED  | No       |                   | FK to seller_products.id                          |
| asset_id    | UUID             | Yes      | NULL              | FK to assets.id (Storage module). NULL if external URL |
| url         | VARCHAR(500)     | No       |                   | Full image or video URL (CDN URL for uploaded, vendor URL for sourced) |
| type        | VARCHAR(10)      | No       | 'image'           | Media type: image or video                        |
| thumbnail_url | VARCHAR(500)   | Yes      | NULL              | Video poster/thumbnail URL. NULL for images.      |
| duration    | INT UNSIGNED     | Yes      | NULL              | Video duration in seconds. NULL for images.       |
| position    | SMALLINT UNSIGNED | No      | 0                 | Display order (0 = first)                         |
| is_primary  | BOOLEAN          | No       | false             | Whether this is the primary/hero image. Only applicable to type=image. |
| created_at  | TIMESTAMP        | No       |                   |                                                   |
| updated_at  | TIMESTAMP        | No       |                   |                                                   |

**Indexes:**

- `PRIMARY KEY (id)`
- `INDEX (product_id, position)` — ordered gallery per product
- `INDEX (asset_id)` — reverse lookup from asset

**Foreign Keys:**

- `product_id REFERENCES seller_products(id) ON DELETE CASCADE`
- `asset_id REFERENCES assets(id) ON DELETE SET NULL`

**Design Decision — Why both `asset_id` and `url`:**
When a seller uploads an image or video through the media center, `asset_id` points to the Storage module asset and `url` is the CDN URL of that asset. When a seller sources from catalog, the media are vendor URLs (e.g., Amazon CDN) with no corresponding asset record — `asset_id` is NULL and `url` holds the vendor URL directly. This dual approach avoids forcing a download-and-reupload of vendor media during product creation.

**Design Decision — Why `is_primary` is a boolean on each row instead of a `primary_image_id` on the product:**
Multiple gallery items (images and videos) belong to one product, and exactly one image should be primary. A boolean on the gallery row keeps the primary designation co-located with the media data and avoids a circular FK (product → gallery → product). The application enforces that exactly one image row per product has `is_primary = true` (videos cannot be primary).

**Design Decision — Why `type` instead of a separate videos table:**
Videos and images share the same ordering, the same asset pipeline, and the same gallery limit (max 20 combined). A `type` discriminator keeps them in one table, avoids join complexity, and allows mixed ordering (e.g., video at position 0, image at position 1).

**Design Decision — `is_primary` applies only to type=image:**
Videos cannot be the primary/hero media. The constraint "exactly one `is_primary = true`" applies only among image rows. This ensures the hero thumbnail for the product is always a static image, which is universally supported by listing UIs and SEO previews.

## seller_product_sku_regions

Per-SKU, per-region pricing, stock, and visibility. Each row represents a SKU's availability in one region.

| Column         | Type             | Nullable | Default        | Description                                                |
| -------------- | ---------------- | -------- | -------------- | ---------------------------------------------------------- |
| id             | BIGINT UNSIGNED  | No       | AUTO_INCREMENT | Primary key                                                |
| sku_id         | BIGINT UNSIGNED  | No       |                | FK to seller_product_skus.id                               |
| region_id      | BIGINT UNSIGNED  | No       |                | FK to regions.id (Core module)                             |
| original_price | DECIMAL(12,2)    | No       |                | Full price in region's currency                            |
| discount_price | DECIMAL(12,2)    | Yes      | NULL           | Sale/discounted price in region's currency. NULL = no discount |
| stock          | INT UNSIGNED     | No       | 0              | Available stock in this region                             |
| is_active      | BOOLEAN          | No       | true           | Whether this SKU is visible/purchasable in this region     |
| created_at     | TIMESTAMP        | No       |                |                                                            |
| updated_at     | TIMESTAMP        | No       |                |                                                            |

**Indexes:**

- `PRIMARY KEY (id)`
- `UNIQUE (sku_id, region_id)` — one price entry per SKU per region
- `INDEX (region_id)` — list by region

**Foreign Keys:**

- `sku_id REFERENCES seller_product_skus(id) ON DELETE CASCADE`
- `region_id REFERENCES regions(id) ON DELETE CASCADE`

**Design Decision — Why currency is NOT stored on this table:**
Currency is derived from `region.currency_id`. This avoids data duplication and stays consistent with the `AddonServicePrice` pattern already established in the codebase. The API response can include the currency by resolving through the region relation.

**Design Decision — Why `discount_price` is nullable:**
NULL means no discount — the `original_price` is the selling price. This is simpler than storing `original_price == discount_price` which would require the client to detect "no discount" by comparing two values.

**Design Decision — Regional vs global activation:**
A SKU is purchasable in a region only when BOTH `seller_product_skus.is_active = true` (global) AND `seller_product_sku_regions.is_active = true` (regional). The global toggle provides a quick way to disable a SKU everywhere, while the regional toggle allows per-market control.

**Design Decision — Regional availability semantics:**
A SKU without any `seller_product_sku_regions` rows is not available in any region. Regional pricing is required for activation — a product can't go active without at least one region-priced SKU.

---

## Enums

### SellerProductStatusEnum

| Value     | Description                                    |
| --------- | ---------------------------------------------- |
| draft     | Incomplete, not visible to buyers              |
| pending   | Submitted for review                           |
| active    | Live, visible to buyers                        |
| inactive  | Taken offline by seller                        |
| violation | Flagged for policy issues                      |
| deleted   | Soft deleted                                   |

### SellerProductSortFieldsEnum

| Value      | Description           |
| ---------- | --------------------- |
| title      | Alphabetical by title |
| created_at | By creation date      |
| updated_at | By last update        |
| price      | By minimum SKU price  |

### SellerProductFiltersEnum

| Value                 | Description                         |
| --------------------- | ----------------------------------- |
| status                | Filter by status enum value         |
| shipping_category_id  | Filter by category UUID             |
| title                 | Search by title (partial match)     |

### SellerProductFieldsEnum

| Value                 | Description                                |
| --------------------- | ------------------------------------------ |
| id                    | BIGINT primary key                         |
| owner_type            | Polymorphic owner class                    |
| owner_id              | Polymorphic owner ID                       |
| shipping_category_id  | Assigned shipping category                 |
| source_product_id     | Linked source product ID                   |
| title                 | Product title                              |
| description           | Product description                        |
| status                | Current status                             |
| created_by            | Creator user ID                            |
| etag                  | ETag hash for concurrency control          |
| lock_version          | Monotonic version counter                  |
| min_price             | Denormalized minimum SKU price             |
| max_price             | Denormalized maximum SKU price             |
| created_at            | Created timestamp                          |
| updated_at            | Updated timestamp                          |

### SellerProductExpandEnum

| Value          | Description                                          |
| -------------- | ---------------------------------------------------- |
| properties     | Include properties with values                       |
| skus           | Include SKU matrix with associations                 |
| specifications | Include specification key-value pairs                |
| gallery        | Include gallery images and videos                    |
| category       | Include shipping category details                    |
| regional_prices | Include per-region pricing/stock/availability on SKUs |

### ResourceObject (additions to existing)

| Value                                     | Description                           |
| ----------------------------------------- | ------------------------------------- |
| SellerProduct                             | Full product detail                   |
| SellerProductMinimal                      | Minimal product (list item)           |
| SellerProductCollection                   | Paginated product list                |
| SellerProductProperty                    | Property axis                         |
| SellerProductPropertyCollection          | Properties list                       |
| SellerProductPropertyValue               | Property value                        |
| SellerProductPropertyValueCollection     | Property values list                  |
| SellerProductSku                          | SKU with price/stock                  |
| SellerProductSkuCollection                | SKU list                              |
| SellerProductSkuAssociation               | SKU-to-value link                     |
| SellerProductSkuAssociationCollection     | SKU associations list                 |
| SellerProductSpecification                | Specification key-value               |
| SellerProductSpecificationCollection      | Specifications list                   |
| SellerProductGalleryItem                  | Gallery item (image or video)         |
| SellerProductGalleryCollection            | Gallery list                          |
| SellerProductStatusCounts                 | Status count breakdown                |
| SellerProductSkuRegionPrice               | SKU regional pricing                  |
| SellerProductSkuRegionPriceCollection     | SKU regional prices list              |
