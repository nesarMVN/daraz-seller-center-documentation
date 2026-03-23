# Technical Overview

**Module:** Seller (new module in `moveon-red`)
**Namespace:** `MoveOn\Seller`

The Seller module is a new Concord module registered via `ModuleServiceProvider`. It owns all seller product data — the product record, properties, property values, SKU matrix, specifications, and gallery. It does not own categories (ShippingCore), assets (Storage), or source products (product-service).

## How It Fits With Existing Modules

| Module              | System        | Relationship                                                                |
| ------------------- | ------------- | --------------------------------------------------------------------------- |
| **ShippingCore**    | moveon-red    | Provides the shipping category tree. Seller product references a leaf `shipping_category_id`. Category ID maps to a schema definition for properties and specifications. |
| **Storage**         | moveon-red    | Manages physical files (S3). Gallery images reference asset UUIDs from Storage. |
| **Core**            | moveon-red    | Provides MediaFolder for asset organization. Also provides IdempotencyService if needed for bulk ops. Provides `regions` and `currencies` tables used for regional pricing. |
| **Store**           | moveon-red    | Seller products use polymorphic ownership (`owner_type` + `owner_id`). For agent-managed sellers, the owner is an `AgentCompany`; for direct sellers, the owner is a `User`. |
| **product-service** | separate service | Read-only vendor catalog. Seller product optionally stores `source_product_id` for traceability and pre-fill. |

---

## Architecture Diagrams

### 1. Module Responsibility and Boundaries

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                    SELLER MODULE                                         │
│                               (Product Management)                                       │
│                                                                                          │
│  ┌──────────────────────────┐    ┌──────────────────────────────────────────────────┐    │
│  │  seller_products          │    │  seller_product_properties                       │    │
│  ├──────────────────────────┤    ├──────────────────────────────────────────────────┤    │
│  │ • id (BIGINT PK)         │    │ • id (BIGINT PK)                                │    │
│  │ • title                  │    │ • product_id (FK → seller_products)              │    │
│  │ • description            │    │ • name                                           │    │
│  │ • status                 │    │ • position                                       │    │
│  │ • shipping_category_id   │    └──────────────────────┬───────────────────────────┘    │
│  │   (FK → ShippingCore)    │                           │                                │
│  │ • source_product_id      │    ┌──────────────────────▼───────────────────────────┐    │
│  │   (→ product-service)    │    │  seller_product_property_values                  │    │
│  │ • owner_type             │    ├──────────────────────────────────────────────────┤    │
│  │ • owner_id               │    │ • id (BIGINT PK)                                │    │
│  │   (polymorphic owner)    │    │                                                 │    │
│  │ • etag                   │    │ • property_id (FK → properties)                 │    │
│  │ • lock_version           │    │ • value, image_url, position                     │    │
│  │ • min_price / max_price  │    └─────────────────────────────────────────────────┘    │
│  └────────┬─────────────────┘                                                            │
│           │                                                                              │
│    ┌──────┼──────────────────┬────────────────────────┬────────────────────┐              │
│    │      │                  │                        │                    │              │
│    ▼      │                  ▼                        ▼                    ▼              │
│  ┌────────┴───────────┐  ┌─────────────────────┐  ┌───────────────┐  ┌─────────────┐    │
│  │ seller_product     │  │ seller_product      │  │ seller_product│  │ seller_     │    │
│  │ _skus              │  │ _sku_associations   │  │ _specs        │  │ product_    │    │
│  ├────────────────────┤  ├─────────────────────┤  ├───────────────┤  │ _gallery    │    │
│  │ • id (BIGINT PK)  │  │ • id (BIGINT PK)   │  │ • id (BIGINT) │  ├─────────────┤    │
│  │ • product_id (FK) │  │ • sku_id (FK)      │  │ • product_id  │  │ • id (BIGINT│    │
│  │ • price           │  │ • property_id (FK) │  │ • name        │  │   PK)       │    │
│  │ • stock           │  │ • property_value_   │  │ • value       │  │ • product_id│    │
│  │ • sku_code        │  │   id (FK)          │  │ • type        │  │ • asset_id  │    │
│  │ • is_active       │  │                     │  │ • position    │  │   (→Storage)│    │
│  └────────┬───────────┘  └─────────────────────┘  └───────┬───────┘  │ • url       │    │
│           │                                               │          │ • position  │    │
│        0..n                                            0..n          │ • is_primary│    │
│           │                                               │          └─────────────┘    │
│  ┌────────▼───────────┐                                                                 │
│  │ seller_product     │                                                                 │
│  │ _sku_regions       │                                                                 │
│  ├────────────────────┤                                                                 │
│  │ • id (BIGINT PK)  │                                                                 │
│  │ • sku_id (FK)     │                                                                 │
│  │ • region_id       │                                                                 │
│  │   (FK → Core)     │                                                                 │
│  │ • original_price  │                                                                 │
│  │ • discount_price  │                                                                 │
│  │ • stock           │                                                                 │
│  │ • is_active       │                                                                 │
│  └────────────────────┘                                                                 │
└─────────────────────────────────────────────────────────────────────────────────────────┘
         │                              │                           │
         │ shipping_category_id FK      │ asset_id FK               │ source_product_id
         │ region_id FK (sku_regions)   │                           │
         ▼                              ▼                           ▼
┌─────────────────┐       ┌──────────────────┐       ┌──────────────────────────┐
│  SHIPPING CORE  │       │  STORAGE MODULE  │       │  PRODUCT-SERVICE         │
│  (Categories)   │       │  (Assets / S3)   │       │  (Read-Only Catalog)     │
│                 │       │                  │       │                          │
│ shipping_       │       │ assets table     │       │ internal_product_read    │
│ categories      │       │ • id (UUID)      │       │ • id (ULID)             │
│ • id (UUID)     │       │ • path           │       │ • product (JSON)        │
│ • name          │       │ • url            │       │                          │
│ • parent_id     │       │                  │       │                          │
└─────────────────┘       └──────────────────┘       └──────────────────────────┘

┌─────────────────────────────────────────────────┐
│                  CORE MODULE                     │
│            (Regions, Currencies)                 │
│                                                  │
│ regions                          currencies      │
│ • id (BIGINT PK)                 • id (PK)       │
│ • name                          • code           │
│ • code (BD/IN/..)               • symbol         │
│ • currency_id (FK)              • name           │
│ • status                                         │
└─────────────────────────────────────────────────┘
```

### 2. Data Flow Between Modules

```
┌───────────┐    ┌──────────────────┐    ┌──────────────────┐    ┌─────────────┐
│  Client   │    │  SELLER MODULE   │    │  SHIPPING CORE   │    │  STORAGE    │
└─────┬─────┘    └────────┬─────────┘    └────────┬─────────┘    └──────┬──────┘
      │                   │                       │                     │
      │  Create Product   │                       │                     │
      │──────────────────►│                       │                     │
      │  (category_id,    │  Validate category    │                     │
      │   properties,     │──────────────────────►│                     │
      │   specs, gallery) │                       │                     │
      │                   │◄──────────────────────│                     │
      │                   │  Category exists ✓     │                     │
      │                   │                       │                     │
      │                   │  Validate asset_ids   │                     │
      │                   │───────────────────────┼────────────────────►│
      │                   │                       │                     │
      │                   │◄──────────────────────┼─────────────────────│
      │                   │  Assets exist ✓        │                     │
      │                   │                       │                     │
      │                   │  Store product +      │                     │
      │                   │  relations in DB      │                     │
      │                   │                       │                     │
      │◄──────────────────│                       │                     │
      │  201 Created      │                       │                     │
      │                   │                       │                     │
      │                   │                       │                     │
┌─────┴─────┐    ┌────────┴─────────┐    ┌───────────────────────────────────┐
│  Client   │    │  SELLER MODULE   │    │       PRODUCT-SERVICE             │
└─────┬─────┘    └────────┬─────────┘    └───────────────┬───────────────────┘
      │                   │                              │
      │  Source from      │                              │
      │  Catalog          │  GET /find/{productId}       │
      │──────────────────►│─────────────────────────────►│
      │                   │                              │
      │                   │◄─────────────────────────────│
      │                   │  Source product data          │
      │                   │                              │
      │◄──────────────────│                              │
      │  Pre-filled form  │                              │
      │                   │                              │
```

### 3. Entity Relationship Diagram

```
┌──────────────────────────┐
│    seller_products        │
├──────────────────────────┤
│ PK  id (BIGINT)          │
│     title                │
│     description          │
│     status               │
│ FK  shipping_category_id │──────────► shipping_categories
│     source_product_id    │- - - - -► product-service (external)
│     owner_type           │
│     owner_id             │─ ─ ─ ─ ─► agent_companies / users (polymorphic)
│     created_by           │
│     etag                 │
│     lock_version         │
│     min_price            │
│     max_price            │
│     deleted_at           │
└────────┬─────────────────┘
         │
         │ 1
         │
    ┌────┼──────────────┬──────────────────┬─────────────────┐
    │    │              │                  │                 │
    │ 0..n           0..n              0..n              0..n
    │    │              │                  │                 │
    ▼    │              ▼                  ▼                 ▼
┌────────┴───────┐ ┌────────────────┐ ┌────────────────┐ ┌──────────────────┐
│ seller_product │ │ seller_product │ │ seller_product │ │ seller_product   │
│ _properties    │ │ _skus          │ │ _specifications│ │ _gallery         │
├────────────────┤ ├────────────────┤ ├────────────────┤ ├──────────────────┤
│ PK id (BIGINT) │ │ PK id (BIGINT) │ │ PK id (BIGINT) │ │ PK id (BIGINT)   │
│ FK product_id  │ │ FK product_id  │ │ FK product_id  │ │ FK product_id    │
│    name        │ │    price       │ │    name        │ │ FK asset_id      │
│    position    │ │    stock       │ │    value       │ │    position      │
│                │ │    sku_code    │ │    type        │ │    is_primary    │
│                │ │    is_active   │ │                │ │                  │
└───────┬────────┘ └───────┬────────┘ └───────┬────────┘ └──────────────────┘
        │                  │                  │
        │ 1                │ 1                │ 1
        │                  │                  │
   ┌────┤               ┌──┤                  │
   │ 0..n            0..n  │               0..n
   │    │                │  │                  │
   │    ▼                │  ▼                  ▼
   │ ┌────────────────┐  │ ┌──────────────────────┐
   │ │ seller_product │  │ │ seller_product       │
   │ │ _property      │  │ │ _sku_associations    │
   │ │ _values        │  │ ├──────────────────────┤
   │ ├────────────────┤  │ │ PK id (BIGINT)       │
   │ │ PK id (BIGINT) │  │ │ FK sku_id            │
   │ │ FK property_id│  │ │ FK property_id      │
   │ │    value       │  │ │ FK property_value_id│
   │ │    image_url   │  │ └──────────────────────┘
   │ │    position    │  │
   │ └────────────────┘  │ 1
   │                     │
   │                     │ 0..n
   │                     │    │
   │                     │    ▼
   │                     │ ┌──────────────────────┐
   │                     │ │ seller_product       │
   │                     │ │ _sku_regions         │
   │                     │ ├──────────────────────┤
   │                     │ │ PK id (BIGINT)       │
   │                     │ │ FK sku_id            │
   │                     │ │ FK region_id (→ Core)│
   │                     │ │    original_price    │
   │                     │ │    discount_price    │
   │                     │ │    stock             │
   │                     │ │    is_active         │
   │                     │ └──────────────────────┘
   │                     │
   └─────────────────────┘

External dependencies:
  region_id  ──────────► regions (Core module)
  region.currency_id ──► currencies (Core module)
```

### 4. Product Creation Flow (Sourced and Manual)

```
                         ┌───────────────────────┐
                         │     Seller Client      │
                         └───────────┬───────────┘
                                     │
                         ┌───────────▼───────────┐
                         │  Choose Creation Mode  │
                         └───────┬───────┬───────┘
                                 │       │
              ┌──────────────────┘       └──────────────────┐
              ▼                                             ▼
   ┌─────────────────────┐                    ┌─────────────────────┐
   │  FLOW A: Source      │                    │  FLOW B: Manual     │
   │  from Catalog        │                    │  Creation           │
   └──────────┬──────────┘                    └──────────┬──────────┘
              │                                          │
              ▼                                          │
   ┌─────────────────────┐                               │
   │  Search product-    │                               │
   │  service by URL     │                               │
   │  or keyword         │                               │
   └──────────┬──────────┘                               │
              │                                          │
              ▼                                          │
   ┌─────────────────────┐                               │
   │  Receive source     │                               │
   │  product data       │                               │
   │  (title, price,     │                               │
   │   properties,       │                               │
   │   specs, images)    │                               │
   └──────────┬──────────┘                               │
              │                                          │
              └──────────────┬───────────────────────────┘
                             │
                             ▼
              ┌──────────────────────────┐
              │  Select Shipping         │
              │  Category (leaf node)    │
              │  from ShippingCore       │
              └──────────────┬───────────┘
                             │
                             ▼
              ┌──────────────────────────┐
              │  Load Category Schema    │
              │  (variation_attributes,  │
              │   specification_         │
              │   attributes)            │
              └──────────────┬───────────┘
                             │
                             ▼
              ┌──────────────────────────┐
              │  Render Dynamic Form     │
              │  ┌────────────────────┐  │
              │  │ Properties Section │  │
              │  │ (schema-driven     │  │
              │  │  axes + values)    │  │
              │  ├────────────────────┤  │
              │  │ SKU Matrix Table   │  │
              │  │ (price, stock per  │  │
              │  │  combination)      │  │
              │  ├────────────────────┤  │
              │  │ Specs Section      │  │
              │  │ (text, select,     │  │
              │  │  required markers) │  │
              │  ├────────────────────┤  │
              │  │ Gallery Section    │  │
              │  │ (upload / pick     │  │
              │  │  from media center)│  │
              │  └────────────────────┘  │
              └──────────────┬───────────┘
                             │
                             ▼
              ┌──────────────────────────┐
              │  POST /seller-products   │
              │                          │
              │  Backend validates:      │
              │  • category exists       │
              │  • properties match      │
              │    schema                │
              │  • specs match schema    │
              │  • SKU associations      │
              │    are consistent        │
              │  • gallery assets exist  │
              └──────────────┬───────────┘
                             │
                             ▼
              ┌──────────────────────────┐
              │  Store in DB             │
              │  (transaction):          │
              │  1. seller_products      │
              │  2. properties           │
              │  3. property_values      │
              │  4. skus                 │
              │  5. sku_associations     │
              │  6. specifications       │
              │  7. gallery              │
              └──────────────────────────┘
```

### 5. Property / SKU Relationship Model

```
Example: T-Shirt with Size and Color properties

Properties (seller_product_properties):
┌────────────┬───────────┬──────────┐
│ id         │ name      │ position │
├────────────┼───────────┼──────────┤
│ 1          │ Size      │ 0        │
│ 2          │ Color     │ 1        │
└────────────┴───────────┴──────────┘

Property Values (seller_product_property_values):
┌────────────┬──────────────┬───────────┬──────────┐
│ id         │ property_id │ value     │ position │
├────────────┼──────────────┼───────────┼──────────┤
│ 1          │ 1            │ S         │ 0        │
│ 2          │ 1            │ M         │ 1        │
│ 3          │ 1            │ L         │ 2        │
│ 4          │ 2            │ Red       │ 0        │
│ 5          │ 2            │ Blue      │ 1        │
└────────────┴──────────────┴───────────┴──────────┘

SKU Matrix (seller_product_skus):
┌────────────┬─────────┬───────┬───────────┐
│ id         │ price   │ stock │ is_active │
├────────────┼─────────┼───────┼───────────┤
│ 101        │ 1200.00 │ 50    │ true      │  ← S + Red
│ 102        │ 1200.00 │ 30    │ true      │  ← S + Blue
│ 103        │ 1300.00 │ 45    │ true      │  ← M + Red
│ 104        │ 1300.00 │ 25    │ true      │  ← M + Blue
│ 105        │ 1400.00 │ 20    │ true      │  ← L + Red
│ 106        │ 1400.00 │ 15    │ false     │  ← L + Blue (disabled)
└────────────┴─────────┴───────┴───────────┘

SKU Associations (seller_product_sku_associations):
┌────────────┬─────────┬──────────────┬─────────────────────┐
│ id         │ sku_id  │ property_id │ property_value_id   │
├────────────┼─────────┼──────────────┼─────────────────────┤
│ 1          │ 101     │ 1 (Size)     │ 1 (S)               │
│ 2          │ 101     │ 2 (Color)    │ 4 (Red)             │
│ 3          │ 102     │ 1 (Size)     │ 1 (S)               │
│ 4          │ 102     │ 2 (Color)    │ 5 (Blue)            │
│ ...        │ ...     │ ...          │ ...                 │
└────────────┴─────────┴──────────────┴─────────────────────┘

How a SKU resolves to its property combination:

  sku 101 ──► assoc 1 ──► value 1 (Size: S)
         └──► assoc 2 ──► value 4 (Color: Red)

  Result: SKU 101 = Size S + Color Red = BDT 1200.00, 50 in stock
```

### 6. Status Lifecycle

```
                              ┌──────────┐
                    ┌────────►│  Active   │◄────────┐
                    │         └────┬──────┘         │
                    │              │                 │
                    │         deactivate        activate
                    │              │                 │
                    │              ▼                 │
  ┌──────────┐  submit   ┌──────────────┐          │
  │  Draft   │──────────►│   Pending    │──────────┘
  └──────────┘  (when     └──────┬──────┘   approve
       │        complete)        │
       │                    flag │ violation
       │                         ▼
       │                  ┌──────────────┐
       │                  │  Violation   │
       │                  └──────┬──────┘
       │                    fix + │ resubmit
       │                    revert│ to pending
       │                         ▼
       │                  ┌──────────────┐
       │                  │   Pending    │
       │                  └──────────────┘
       │
       │         ┌──────────────┐
       │         │  Inactive    │
       │         └──────┬──────┘
       │                │
       │    delete       │  delete
       │    (from any    │
       │     status)     │
       ▼                 ▼
  ┌──────────────────────────┐
  │         Deleted           │
  │    (soft delete)          │
  └──────────────────────────┘
```
