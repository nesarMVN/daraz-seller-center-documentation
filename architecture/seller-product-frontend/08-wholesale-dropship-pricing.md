# Wholesale & Dropship Pricing

## Overview

This document defines wholesale (tiered) and dropship (flat) pricing for seller products, modeled after 1688's actual API structure as consumed by our own `ProductDetailABCNOpenAPIToV1` converter in `moveon-product-service`.

**Key principle:** Wholesale pricing is **product-level** (global across all SKUs). Dropship pricing is **product-level with SKU-level overrides**. This is how 1688 does it, confirmed from our own integration code.

---

## 1688 API: Actual Data Structure (from our converter code)

Source: `moveon-product-service/modules/MoveOn/Product/src/Converters/ProductDetailABCNOpenAPIToV1.php`

### Product-Level Fields

```
productSaleInfo: {
    priceRangeList: [                          // WHOLESALE TIERS (product-level)
        { startQuantity: 2, price: "68.00" },
        { startQuantity: 50, price: "58.00" },
        { startQuantity: 200, price: "52.00" }
    ],
    amountOnSale: 5000,                        // total stock
    consignPrice: "68.00",                     // consignment/base price
    fenxiaoSaleInfo: {                         // DROPSHIP (product-level)
        onePiecePrice: "82.00",                // dropship single-piece price
        offerPrice: "78.00",                   // dropship offer/promo price
        onePieceFreePostage: true,             // free domestic shipping
        startQuantity: 1                       // dropship MOQ (always 1)
    }
}
minOrderQuantity: 2                            // wholesale MOQ
batchNumber: 1                                 // lot/batch size
```

### SKU-Level Fields

```
productSkuInfos: [
    {
        skuId: "5001234567890",
        price: "72.00",                        // this SKU's unit price
        jxhyPrice: null,                       // premium member price (fallback)
        consignPrice: "68.00",                 // consignment price (fallback)
        amountOnSale: 500,                     // SKU stock
        fenxiaoPriceInfo: {                    // DROPSHIP (SKU-level override)
            onePiecePrice: "85.00",            // may differ from product-level
            offerPrice: "80.00"
        },
        skuAttributes: [...]
    }
]
```

### What Lives Where

```
Product
 |
 +-- price
 |    +-- original          { min, max }       <-- range across all SKUs
 |    +-- discount          { min, max }       <-- range across all SKUs
 |    +-- wholesales[]                         <-- PRODUCT-LEVEL (global)
 |    |    +-- qty_from     2
 |    |    +-- original     68.00
 |    |    +-- batch        1
 |    +-- dropship                             <-- PRODUCT-LEVEL base
 |         +-- available         true/false
 |         +-- one_piece_price   82.00
 |         +-- offer_price       78.00
 |         +-- free_postage      true
 |         +-- start_quantity    1
 |
 +-- variation
      +-- skus[]
           +-- price
                +-- original      72.00        <-- THIS SKU's unit price
                +-- wholesales    []           <-- ALWAYS EMPTY (wholesale = product-level)
                +-- dropship                   <-- SKU-LEVEL override
                     +-- available        true/false
                     +-- one_piece_price  85.00    <-- from sku.fenxiaoPriceInfo
                     +-- offer_price      80.00    <-- from sku.fenxiaoPriceInfo
                     +-- free_postage     true     <-- INHERITED from product-level
                     +-- start_quantity   1        <-- INHERITED from product-level
```

### SKU Price Resolution (priority order from converter)

```
1. sku['price']                              -- standard SKU price
2. sku['jxhyPrice']                          -- premium member price
3. productSaleInfo['priceRangeList'][0]       -- first wholesale tier
4. sku['consignPrice']                        -- consignment price
```

### Wholesale Inclusion Logic (from `getWholesalePrice()`)

Wholesale tiers are included when:
- 2 or more tiers exist, OR
- exactly 1 tier with `startQuantity > 1` AND `moq > 1`

If only 1 tier at qty=1, it's treated as a regular price (not wholesale).

---

## Current Seller Product State

### What Exists (backend + frontend)

**Database tables:**
- `seller_products` -- has `min_price`, `max_price` (denormalized from SKUs)
- `seller_product_skus` -- has `price` (DECIMAL 12,2), `stock`, `is_active`
- `seller_product_sku_regions` -- has `original_price`, `discount_price`, `stock`, `is_active`

**Frontend schemas:**
- `skuFormSchema` -- `price: number`, `stock: number`, `regional_prices: RegionalPriceFormData[]`
- `regionalPriceSchema` -- `region_id`, `original_price`, `discount_price`, `stock`, `is_active`

**JSON Patch paths (backend):**
- `/skus/{id}/price` -- replace SKU base price
- `/skus/{id}/regional_prices/-` -- add regional price
- `/skus/{id}/regional_prices/{id}/original_price` -- replace

### What Does NOT Exist

- No wholesale tier pricing (no quantity breaks anywhere)
- No dropship pricing (no `fenxiao`/`one_piece_price` fields)
- No product-level pricing fields (all pricing is SKU-level)
- `Region.store_type` exists in the Region type but is unused by seller-product

---

## Proposed Schema Changes

### New Tables

#### `seller_product_wholesale_tiers` (product-level)

```
+---------------------+-------------------+----------+---------+----------------------------------+
| Column              | Type              | Nullable | Default | Description                      |
+---------------------+-------------------+----------+---------+----------------------------------+
| id                  | BIGINT PK         | No       | AUTO    |                                  |
| product_id          | FK seller_products| No       |         | One product has many tiers       |
| min_qty             | INT UNSIGNED      | No       |         | Start quantity for this tier     |
| price               | DECIMAL(12,2)     | No       |         | Per-unit price at this tier      |
| position            | SMALLINT UNSIGNED | No       | 0       | Display order                    |
| created_at          | TIMESTAMP         | No       |         |                                  |
| updated_at          | TIMESTAMP         | No       |         |                                  |
+---------------------+-------------------+----------+---------+----------------------------------+

Indexes:
  UNIQUE (product_id, min_qty)   -- no duplicate qty thresholds
  INDEX  (product_id, position)  -- ordered display

FK: product_id -> seller_products(id) ON DELETE CASCADE
```

#### `seller_product_dropship_config` (product-level, 1 row per product)

```
+---------------------+-------------------+----------+---------+----------------------------------+
| Column              | Type              | Nullable | Default | Description                      |
+---------------------+-------------------+----------+---------+----------------------------------+
| id                  | BIGINT PK         | No       | AUTO    |                                  |
| product_id          | FK seller_products| No       |         | UNIQUE -- one config per product |
| enabled             | BOOLEAN           | No       | false   | Dropship opt-in toggle           |
| one_piece_price     | DECIMAL(12,2)     | Yes      | NULL    | Base dropship price              |
| offer_price         | DECIMAL(12,2)     | Yes      | NULL    | Promo/negotiated dropship price  |
| free_postage        | BOOLEAN           | No       | false   | Shipping included in price       |
| start_quantity      | INT UNSIGNED      | No       | 1       | Min qty (typically 1)            |
| created_at          | TIMESTAMP         | No       |         |                                  |
| updated_at          | TIMESTAMP         | No       |         |                                  |
+---------------------+-------------------+----------+---------+----------------------------------+

Indexes:
  UNIQUE (product_id)  -- exactly one config per product

FK: product_id -> seller_products(id) ON DELETE CASCADE
```

#### `seller_product_sku_dropship_prices` (SKU-level overrides)

```
+---------------------+-------------------+----------+---------+----------------------------------+
| Column              | Type              | Nullable | Default | Description                      |
+---------------------+-------------------+----------+---------+----------------------------------+
| id                  | BIGINT PK         | No       | AUTO    |                                  |
| sku_id              | FK skus           | No       |         | UNIQUE -- one override per SKU   |
| one_piece_price     | DECIMAL(12,2)     | No       |         | SKU-specific dropship price      |
| offer_price         | DECIMAL(12,2)     | Yes      | NULL    | SKU-specific promo price         |
| created_at          | TIMESTAMP         | No       |         |                                  |
| updated_at          | TIMESTAMP         | No       |         |                                  |
+---------------------+-------------------+----------+---------+----------------------------------+

Indexes:
  UNIQUE (sku_id)  -- one override per SKU

FK: sku_id -> seller_product_skus(id) ON DELETE CASCADE
```

### Changes to Existing Tables

#### `seller_products` -- add wholesale fields

```
+---------------------+-------------------+----------+---------+----------------------------------+
| Column              | Type              | Nullable | Default | Description                      |
+---------------------+-------------------+----------+---------+----------------------------------+
| wholesale_enabled   | BOOLEAN           | No       | false   | Wholesale pricing opt-in         |
| wholesale_moq       | INT UNSIGNED      | Yes      | NULL    | Minimum order quantity           |
| wholesale_batch     | INT UNSIGNED      | No       | 1       | Batch/lot size                   |
+---------------------+-------------------+----------+---------+----------------------------------+
```

These live on the product because wholesale config is **product-level** (matching 1688).

---

## Section Layout

The pricing section in the product form adds two new tabs above the existing regional pricing:

```
+========================================================================+
|  PRICING                                                               |
|                                                                        |
|  [Wholesale]  [Dropship]  [Regional]                                   |
|  ~~~~~~~~~~~                                                           |
+========================================================================+
```

The existing "Regional Pricing" step becomes the third tab. Wholesale and Dropship are new.

### Wholesale Tab

```
+========================================================================+
|  WHOLESALE PRICING                                                     |
+------------------------------------------------------------------------+
|                                                                        |
|  [x] Enable wholesale pricing                                         |
|                                                                        |
|  MOQ: [ 2 ]       Batch size: [ 1 ]                                   |
|                                                                        |
|  Quantity Tiers:                                                       |
|  +----------+---------------+--------+                                 |
|  | Min Qty  | Price / unit  |        |                                 |
|  +----------+---------------+--------+                                 |
|  |    2     |     68.00     |  [x]   |                                 |
|  |   50     |     58.00     |  [x]   |                                 |
|  |  200     |     52.00     |  [x]   |                                 |
|  +----------+---------------+--------+                                 |
|  [+ Add tier]                          (max 6 tiers)                   |
|                                                                        |
|  Note: These tiers apply to the total order across all SKUs.           |
|  Individual SKU base prices are set in the Properties & SKUs section.  |
|                                                                        |
+------------------------------------------------------------------------+
```

### Dropship Tab

```
+========================================================================+
|  DROPSHIP PRICING                                                      |
+------------------------------------------------------------------------+
|                                                                        |
|  [x] Enable dropship pricing                                          |
|                                                                        |
|  Product Base:                                                         |
|  +-------------------------------+                                     |
|  | One-piece price: [  82.00  ]  |                                     |
|  | Offer price:     [  78.00  ]  |  (optional)                         |
|  | [x] Free postage              |                                     |
|  | Start quantity:  [    1    ]  |                                     |
|  +-------------------------------+                                     |
|                                                                        |
|  SKU Overrides (optional):                                             |
|  Empty = inherits product base price                                   |
|  +-------------------+--------------+--------------+                   |
|  | SKU               | 1-pc Price   | Offer Price  |                   |
|  +-------------------+--------------+--------------+                   |
|  | Red / XL          |    85.00     |    80.00     |                   |
|  | Blue / M          |     --       |     --       |  (uses base)      |
|  | Black / L         |    88.00     |     --       |                   |
|  +-------------------+--------------+--------------+                   |
|                                                                        |
+------------------------------------------------------------------------+
```

### Wholesale Tab Interaction States

```
+========================================================================+
|  WHOLESALE TOGGLE STATES                                               |
+------------------------------------------------------------------------+
|                                                                        |
|  DISABLED (wholesale_enabled = false):                                 |
|  +----------------------------------------------------------+         |
|  | [ ] Enable wholesale pricing                              |         |
|  |                                                            |         |
|  | (all fields hidden, grayed-out area)                      |         |
|  +----------------------------------------------------------+         |
|                                                                        |
|  ENABLED, NO TIERS:                                                    |
|  +----------------------------------------------------------+         |
|  | [x] Enable wholesale pricing                              |         |
|  |                                                            |         |
|  | MOQ: [ 2 ]     Batch: [ 1 ]                               |         |
|  |                                                            |         |
|  | Quantity Tiers:                                            |         |
|  | (empty state)                                              |         |
|  | "Add at least 2 tiers to enable wholesale pricing"         |         |
|  |                                                            |         |
|  | [+ Add tier]                                               |         |
|  +----------------------------------------------------------+         |
|                                                                        |
|  ADD TIER:                                                             |
|  +----------------------------------------------------------+         |
|  | Clicking [+ Add tier] inserts a new row:                   |         |
|  | +----------+-----------+--------+                          |         |
|  | |    2     |   68.00   |  [x]   | <- existing             |         |
|  | |   50     |   58.00   |  [x]   | <- existing             |         |
|  | | [     ]  | [       ] |  [x]   | <- new (focus on qty)   |         |
|  | +----------+-----------+--------+                          |         |
|  |                                                            |         |
|  | New tier's min_qty must be > previous tier's min_qty.      |         |
|  | New tier's price should be <= previous tier's price.       |         |
|  +----------------------------------------------------------+         |
|                                                                        |
|  REMOVE TIER:                                                          |
|  Click [x] on a tier row removes it.                                   |
|  No confirmation dialog (simple remove).                               |
|  Cannot remove if only 2 tiers remain (minimum 2 for wholesale).      |
|                                                                        |
+========================================================================+
```

### Dropship Tab Interaction States

```
+========================================================================+
|  DROPSHIP TOGGLE STATES                                                |
+------------------------------------------------------------------------+
|                                                                        |
|  DISABLED (enabled = false):                                           |
|  +----------------------------------------------------------+         |
|  | [ ] Enable dropship pricing                               |         |
|  |                                                            |         |
|  | (all fields hidden)                                        |         |
|  +----------------------------------------------------------+         |
|                                                                        |
|  ENABLED, NO SKUS (product has no SKUs yet):                           |
|  +----------------------------------------------------------+         |
|  | [x] Enable dropship pricing                               |         |
|  |                                                            |         |
|  | Product Base:                                              |         |
|  | One-piece price: [       ]                                 |         |
|  | Offer price:     [       ]  (optional)                     |         |
|  | [ ] Free postage                                           |         |
|  | Start quantity:  [  1  ]                                   |         |
|  |                                                            |         |
|  | SKU Overrides:                                             |         |
|  | "Create SKUs first to set per-SKU dropship overrides"      |         |
|  +----------------------------------------------------------+         |
|                                                                        |
|  SKU OVERRIDE ROW:                                                     |
|  +-------------------+--------------+--------------+                   |
|  | Red / XL          | [  85.00  ]  | [  80.00  ]  |                   |
|  +-------------------+--------------+--------------+                   |
|  Clearing both fields = SKU inherits product base price.               |
|  offer_price must be <= one_piece_price (warning, not block).          |
|                                                                        |
+========================================================================+
```

---

## Validation Rules

### Wholesale Tiers

```
+---------------------------------------------+--------------------------------------+
| Rule                                        | Client Error                         |
+---------------------------------------------+--------------------------------------+
| wholesale_enabled required                  | (toggle, always present)             |
| wholesale_moq >= 1 (when enabled)           | "MOQ must be at least 1"             |
| wholesale_batch >= 1                        | "Batch size must be at least 1"      |
| At least 2 tiers when enabled               | "Add at least 2 quantity tiers"      |
| Max 6 tiers                                 | "Maximum 6 tiers allowed"            |
| tier.min_qty >= 1                           | "Quantity must be at least 1"        |
| tier.min_qty unique across tiers            | "Duplicate quantity threshold"        |
| tier.min_qty ascending order                | "Tiers must be in ascending order"   |
| tier.price >= 0.01                          | "Price must be at least 0.01"        |
| tier.price <= 9999999999.99                 | "Price exceeds maximum"              |
| tier.price should decrease as qty increases | (warning, not hard error)            |
| first tier min_qty >= moq                   | "First tier must start at MOQ or above"|
+---------------------------------------------+--------------------------------------+
```

### Dropship Config

```
+---------------------------------------------+--------------------------------------+
| Rule                                        | Client Error                         |
+---------------------------------------------+--------------------------------------+
| enabled required                            | (toggle, always present)             |
| one_piece_price >= 0.01 (when enabled)      | "Dropship price is required"         |
| one_piece_price <= 9999999999.99            | "Price exceeds maximum"              |
| offer_price >= 0.01 (when set)              | "Offer price must be at least 0.01"  |
| offer_price <= one_piece_price (when set)   | (warning: "Offer exceeds base")      |
| start_quantity >= 1                         | "Start quantity must be at least 1"  |
+---------------------------------------------+--------------------------------------+
```

### SKU Dropship Overrides

```
+---------------------------------------------+--------------------------------------+
| Rule                                        | Client Error                         |
+---------------------------------------------+--------------------------------------+
| one_piece_price >= 0.01 (when set)          | "Price must be at least 0.01"        |
| offer_price >= 0.01 (when set)              | "Offer price must be at least 0.01"  |
| offer_price <= one_piece_price (when set)   | (warning, not block)                 |
| Both empty = inherits product base          | (valid, shows "(base)" label)        |
+---------------------------------------------+--------------------------------------+
```

---

## API Changes

### Create Product (POST)

New fields in request body:

```json
{
  "wholesale_enabled": true,
  "wholesale_moq": 2,
  "wholesale_batch": 1,
  "wholesale_tiers": [
    { "min_qty": 2, "price": 68.00, "position": 0 },
    { "min_qty": 50, "price": 58.00, "position": 1 },
    { "min_qty": 200, "price": 52.00, "position": 2 }
  ],
  "dropship": {
    "enabled": true,
    "one_piece_price": 82.00,
    "offer_price": 78.00,
    "free_postage": true,
    "start_quantity": 1
  },
  "skus": [
    {
      "price": 72.00,
      "stock": 500,
      "property_values": [[0, 0], [1, 0]],
      "dropship_override": {
        "one_piece_price": 85.00,
        "offer_price": 80.00
      }
    },
    {
      "price": 68.00,
      "stock": 300,
      "property_values": [[0, 1], [1, 0]],
      "dropship_override": null
    }
  ]
}
```

### Show Product (GET /{id})

New expand values:
- `expand[]=wholesale_tiers` -- include wholesale tier data
- `expand[]=dropship` -- include dropship config + SKU overrides

Response shape:

```json
{
  "object": "SellerProduct",
  "id": 1,
  "wholesale_enabled": true,
  "wholesale_moq": 2,
  "wholesale_batch": 1,
  "wholesale_tiers": {
    "object": "SellerProductWholesaleTierCollection",
    "data": [
      {
        "object": "SellerProductWholesaleTier",
        "id": 1,
        "min_qty": 2,
        "price": "68.00",
        "position": 0
      },
      {
        "object": "SellerProductWholesaleTier",
        "id": 2,
        "min_qty": 50,
        "price": "58.00",
        "position": 1
      }
    ]
  },
  "dropship": {
    "object": "SellerProductDropshipConfig",
    "id": 1,
    "enabled": true,
    "one_piece_price": "82.00",
    "offer_price": "78.00",
    "free_postage": true,
    "start_quantity": 1
  },
  "skus": {
    "data": [
      {
        "object": "SellerProductSku",
        "id": 101,
        "price": "72.00",
        "stock": 500,
        "dropship_override": {
          "object": "SellerProductSkuDropshipPrice",
          "id": 1,
          "one_piece_price": "85.00",
          "offer_price": "80.00"
        }
      },
      {
        "object": "SellerProductSku",
        "id": 102,
        "price": "68.00",
        "stock": 300,
        "dropship_override": null
      }
    ]
  }
}
```

### Update Product (PATCH) -- New JSON Patch Paths

#### Wholesale Paths

```
Replace:
  /wholesale_enabled                          boolean
  /wholesale_moq                              integer (>= 1)
  /wholesale_batch                            integer (>= 1)
  /wholesale_tiers/{id}/min_qty               integer (>= 1)
  /wholesale_tiers/{id}/price                 number  (>= 0.01)
  /wholesale_tiers/{id}/position              integer

Add:
  /wholesale_tiers/-                          object { min_qty, price, position }

Remove:
  /wholesale_tiers/{id}
```

#### Dropship Paths

```
Replace:
  /dropship/enabled                           boolean
  /dropship/one_piece_price                   number (>= 0.01) or null
  /dropship/offer_price                       number (>= 0.01) or null
  /dropship/free_postage                      boolean
  /dropship/start_quantity                    integer (>= 1)

SKU dropship override:
  /skus/{id}/dropship_override                object or null (add/replace entire)

Replace individual SKU override fields:
  /skus/{id}/dropship_override/one_piece_price    number (>= 0.01)
  /skus/{id}/dropship_override/offer_price        number (>= 0.01) or null

Add SKU dropship override:
  /skus/{id}/dropship_override                object { one_piece_price, offer_price? }

Remove SKU dropship override:
  /skus/{id}/dropship_override                (sets to null = inherits base)
```

### List Products (GET /)

`price_range` in list response is unchanged -- still based on SKU base prices. Wholesale and dropship are not reflected in list price ranges.

---

## Data Flow

### Create Flow

```
[Seller fills form]
       |
       v
submitService.buildCreateProductData()
  +-- existing: title, description, category, properties, skus, specs, gallery, regional_prices
  +-- NEW: wholesale_enabled, wholesale_moq, wholesale_batch, wholesale_tiers[]
  +-- NEW: dropship { enabled, one_piece_price, offer_price, free_postage, start_quantity }
  +-- NEW: skus[].dropship_override { one_piece_price, offer_price }
       |
       v
POST /seller-products
       |
       v
Backend: transaction
  1. Create seller_product row (with wholesale_enabled, wholesale_moq, wholesale_batch)
  2. Create wholesale_tiers rows (if enabled)
  3. Create dropship_config row (if enabled)
  4. Create SKUs (existing flow)
  5. Create SKU dropship overrides (if any)
  6. computeAndStore() etag
```

### Edit Flow (JSON Patch)

```
[Seller changes wholesale tier price]
       |
       v
editPatchService builds patch op:
  { op: "replace", path: "/wholesale_tiers/42/price", value: 55.00 }
       |
       v
PATCH /seller-products/{id}
  If-Match: "{etag}"
  Content-Type: application/json-patch+json
       |
       v
Backend: PatchPathValidator allows the path
  -> PatchOperationBuilder creates db-op
  -> PatchDbExecutor delegates to SellerProductWholesaleTierService
  -> Post-patch validation: min 2 tiers, ascending min_qty, etc.
  -> Recompute etag
```

### Preview Price Display

```
previewService.getSkuPriceDisplay():
  IF regional pricing selected -> show regional price (existing)
  ELSE IF dropship tab active  -> show dropship price (SKU override or product base)
  ELSE                         -> show SKU base price + wholesale tiers badge
```

---

## Frontend Schema Changes

### New Zod Schemas

```ts
// Wholesale tier (product-level)
const wholesaleTierSchema = z.object({
    min_qty: z.coerce.number().int().min(1),
    price: z.coerce.number().min(0.01).max(9999999999.99),
    position: z.coerce.number().int().min(0),
})

// Dropship config (product-level)
const dropshipConfigSchema = z.object({
    enabled: z.boolean(),
    one_piece_price: z.coerce.number().min(0.01).max(9999999999.99).nullable(),
    offer_price: z.coerce.number().min(0.01).max(9999999999.99).nullable().optional(),
    free_postage: z.boolean(),
    start_quantity: z.coerce.number().int().min(1),
})

// SKU dropship override (per-SKU, optional)
const skuDropshipOverrideSchema = z.object({
    one_piece_price: z.coerce.number().min(0.01).max(9999999999.99),
    offer_price: z.coerce.number().min(0.01).max(9999999999.99).nullable().optional(),
}).nullable()
```

### Extended Product Schema

```ts
const createSellerProductSchema = z.object({
    // ... existing fields ...

    // NEW: Wholesale (product-level)
    wholesale_enabled: z.boolean(),
    wholesale_moq: z.coerce.number().int().min(1).nullable(),
    wholesale_batch: z.coerce.number().int().min(1),
    wholesale_tiers: z.array(wholesaleTierSchema),

    // NEW: Dropship (product-level)
    dropship: dropshipConfigSchema,
})
```

### Extended SKU Schema

```ts
const skuFormSchema = z.object({
    // ... existing fields (price, stock, sku_code, is_active, property_values, regional_prices) ...

    // NEW: Dropship override (per-SKU, optional)
    dropship_override: skuDropshipOverrideSchema.optional(),
})
```

---

## Frontend Types

### New API Response Types

```ts
type SellerProductWholesaleTier = {
    object: 'SellerProductWholesaleTier'
    id: number
    min_qty: number
    price: string           // DECIMAL as string from API
    position: number
}

type SellerProductDropshipConfig = {
    object: 'SellerProductDropshipConfig'
    id: number
    enabled: boolean
    one_piece_price: string | null
    offer_price: string | null
    free_postage: boolean
    start_quantity: number
}

type SellerProductSkuDropshipPrice = {
    object: 'SellerProductSkuDropshipPrice'
    id: number
    one_piece_price: string
    offer_price: string | null
}
```

### Extended Existing Types

```ts
// SellerProduct gains:
type SellerProduct = {
    // ... existing fields ...
    wholesale_enabled: boolean
    wholesale_moq: number | null
    wholesale_batch: number
    wholesale_tiers?: {
        object: 'SellerProductWholesaleTierCollection'
        data: SellerProductWholesaleTier[]
    }
    dropship?: SellerProductDropshipConfig
}

// SellerProductSku gains:
type SellerProductSku = {
    // ... existing fields ...
    dropship_override?: SellerProductSkuDropshipPrice | null
}

// SellerProductMinimal gains (list view):
type SellerProductMinimal = {
    // ... existing fields ...
    wholesale_enabled: boolean
    dropship_enabled: boolean
}
```

---

## Open Questions

1. **Backend API contract finalization** -- The proposed schema and API changes above need backend team review before frontend implementation starts.

2. **Regional pricing interaction** -- Should wholesale/dropship pricing be region-aware? Currently regional pricing is SKU-level. If a product has both wholesale tiers AND regional pricing, which takes precedence? Proposal: keep them independent in v1.

3. **Currency** -- Wholesale tiers and dropship prices -- what currency are they in? The SKU base `price` is in the product's default currency. Wholesale/dropship should follow the same currency.

4. **List view display** -- Should the product list show wholesale/dropship status badges? The `SellerProductMinimal` response could include `wholesale_enabled` and `dropship_enabled` booleans for badge rendering.

5. **Batch helpers** -- For dropship SKU overrides: "Set all to base + X%" utility? For wholesale tiers: "Copy tiers from another product" utility?
