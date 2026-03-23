# Regional Pricing

## Overview

Regional pricing allows sellers to set different prices, stock levels, and availability per SKU for each of the 6 supported regions. Each SKU can have 0-6 regional price entries. A SKU is purchasable in a region only when both `skus.is_active = true` AND `sku_regions.is_active = true`.

**Region Source:**
Region list comes from the Core API `regions` table (not hardcoded). The backend validates
`region_id` with `exists:regions,id`. Currency code (BDT, INR, AED, etc.) is derived from
`region.currency_id` — it is NOT stored on the `seller_product_sku_regions` table.

Currently available regions (may change as Core adds more):
- BD (Bangladesh, BDT)
- IN (India, INR)
- AE (United Arab Emirates, AED)
- US (United States, USD)
- EU (European Union, EUR)
- AU (Australia, AUD)

The frontend should fetch the region list from Core API and cache it, rather than
hardcoding region names/codes/currencies.

**Constraints:**
- One entry per region per SKU (unique constraint: `sku_id, region_id`)
- `original_price`: DECIMAL(12,2), min 0.01, max 9999999999.99
- `discount_price`: DECIMAL(12,2), nullable, min 0.01 when set, must not exceed `original_price`
- `stock`: INT UNSIGNED, min 0
- `is_active`: boolean, default true
- No duplicate `region_id` within a single SKU

---

## Section Layout

The regional pricing section appears below the SKU matrix in the product form. It is organized per-SKU, with each SKU expandable to show its regional price entries.

```
╔════════════════════════════════════════════════════════════════════════╗
║  REGIONAL PRICING                                                      ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Configure region-specific pricing and stock for each SKU.             ║
║  SKUs without regional pricing use the base price/stock.               ║
║                                                                        ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │                                                                  │ ║
║  │  v SKU: Black / S  (Base: $24.99, Stock: 100)          [2 rgns] │ ║
║  │  ┌────────────────────────────────────────────────────────────┐  │ ║
║  │  │                                                            │  │ ║
║  │  │ ┌────────┬──────────────┬──────────────┬───────┬────────┐ │  │ ║
║  │  │ │ Region │ Orig. Price  │ Disc. Price  │ Stock │ Active │ │  │ ║
║  │  │ ├────────┼──────────────┼──────────────┼───────┼────────┤ │  │ ║
║  │  │ │ BD     │ BDT 2,499.00│ BDT 2,199.00│ 50    │ [on]   │ │  │ ║
║  │  │ │        │              │              │       │  [x]   │ │  │ ║
║  │  │ ├────────┼──────────────┼──────────────┼───────┼────────┤ │  │ ║
║  │  │ │ IN     │ INR 2,099.00│ ____________ │ 30    │ [on]   │ │  │ ║
║  │  │ │        │              │              │       │  [x]   │ │  │ ║
║  │  │ └────────┴──────────────┴──────────────┴───────┴────────┘ │  │ ║
║  │  │                                                            │  │ ║
║  │  │ [+ Add Region]                                             │  │ ║
║  │  │ Available: AE, US, EU, AU                                  │  │ ║
║  │  │                                                            │  │ ║
║  │  └────────────────────────────────────────────────────────────┘  │ ║
║  │                                                                  │ ║
║  │  > SKU: Black / M  (Base: $24.99, Stock: 100)          [0 rgns] │ ║
║  │    (click to expand and add regional pricing)                    │ ║
║  │                                                                  │ ║
║  │  > SKU: Black / L  (Base: $26.99, Stock: 80)           [1 rgn]  │ ║
║  │                                                                  │ ║
║  │  > SKU: White / S  (Base: $24.99, Stock: 100)          [0 rgns] │ ║
║  │                                                                  │ ║
║  │  ...                                                             │ ║
║  │                                                                  │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║                                                                        ║
║  Quick setup:                                                          ║
║  ┌────────────────────────────────────────────────────────────────┐   ║
║  │  [Copy BD pricing to all SKUs]  [Copy IN pricing to all SKUs] │   ║
║  └────────────────────────────────────────────────────────────────┘   ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Workflows

### Add Regional Price to a SKU

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: User expands a SKU and clicks [+ Add Region]                ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Region selector dropdown:                                             ║
║  ┌─────────────────────────┐                                          ║
║  │  Select region:          │                                          ║
║  │  ┌─────────────────────┐│                                          ║
║  │  │  BD - Bangladesh    ││  <- already added (disabled)              ║
║  │  │  IN - India         ││  <- already added (disabled)              ║
║  │  │  AE - UAE           ││  <- available                             ║
║  │  │  US - USA           ││  <- available                             ║
║  │  │  EU - European Union││  <- available                             ║
║  │  │  AU - Australia     ││  <- available                             ║
║  │  └─────────────────────┘│                                          ║
║  └─────────────────────────┘                                          ║
║                                                                        ║
║  User selects "AE - UAE":                                              ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Region Dropdown States

```
╔════════════════════════════════════════════════════════════════════════╗
║  REGION SELECTOR INTERACTION STATES                                   ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  CLOSED:                                                               ║
║  ┌─────────────────────────────────┐                                  ║
║  │  + Add Region               v   │                                  ║
║  └─────────────────────────────────┘                                  ║
║  (chevron v, outline button style)                                     ║
║                                                                        ║
║  OPEN (dropdown expanded):                                             ║
║  ┌─────────────────────────────────┐                                  ║
║  │  + Add Region               ^   │                                  ║
║  ├─────────────────────────────────┤                                  ║
║  │  BD - Bangladesh    * Configured│  <- grayed out, already added     ║
║  │  IN - India         * Configured│  <- grayed out, already added     ║
║  │  AE - UAE                       │  <- available (normal text)       ║
║  │  US - USA                       │  <- available                     ║
║  │  EU - European Union            │  <- available                     ║
║  │  AU - Australia                 │  <- available                     ║
║  └─────────────────────────────────┘                                  ║
║  (chevron ^, already-configured items show * and gray text)            ║
║                                                                        ║
║  HOVER on available item:                                              ║
║  │  AE - UAE                       │  <- blue-50 bg highlight          ║
║                                                                        ║
║  On SELECT:                                                            ║
║  - Dropdown closes                                                     ║
║  - New region row animates in (slide-down, 200ms)                      ║
║  - Focus moves to the first empty price field in the new row           ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

```
                                    │
                                    ▼
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 2: New region row appears with empty fields                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌────────┬──────────────┬──────────────┬───────┬────────┐           ║
║  │ Region │ Orig. Price  │ Disc. Price  │ Stock │ Active │           ║
║  ├────────┼──────────────┼──────────────┼───────┼────────┤           ║
║  │ BD     │ BDT 2,499.00│ BDT 2,199.00│ 50    │ [on]   │           ║
║  │ IN     │ INR 2,099.00│ ____________ │ 30    │ [on]   │           ║
║  │ AE     │ [__________]│ [__________] │ [___] │ [on]   │  NEW      ║
║  │        │              │              │       │  [x]   │           ║
║  └────────┴──────────────┴──────────────┴───────┴────────┘           ║
║                                                                        ║
║  User fills in: original_price=149.00, stock=25                        ║
║  discount_price is optional.                                           ║
║                                                                        ║
║  For create mode:                                                      ║
║  Added to the SKU's regional_prices array in the request body.         ║
║                                                                        ║
║  For edit mode, generates patch op:                                    ║
║  {                                                                     ║
║    op: "add",                                                          ║
║    path: "/skus/{skuId}/regional_prices/-",                            ║
║    value: {                                                            ║
║      region_id: 3,                                                     ║
║      original_price: 149.00,                                           ║
║      discount_price: null,                                             ║
║      stock: 25,                                                        ║
║      is_active: true                                                   ║
║    }                                                                   ║
║  }                                                                     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Edit Regional Price

```
╔════════════════════════════════════════════════════════════════════════╗
║  User changes the BD original price from 2499 to 2599                 ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌────────┬──────────────┬──────────────┬───────┬────────┐           ║
║  │ BD     │ [BDT 2599.00]│ BDT 2,199.00│ 50    │ [on]   │           ║
║  │        │ ^^^^ edited  │              │       │        │           ║
║  └────────┴──────────────┴──────────────┴───────┴────────┘           ║
║                                                                        ║
║  For edit mode, generates patch op:                                    ║
║  {                                                                     ║
║    op: "replace",                                                      ║
║    path: "/skus/{skuId}/regional_prices/{regionPriceId}/original_price",║
║    value: 2599.00                                                      ║
║  }                                                                     ║
║                                                                        ║
║  Each editable field generates its own replace op:                      ║
║  - /skus/{id}/regional_prices/{id}/original_price                      ║
║  - /skus/{id}/regional_prices/{id}/discount_price  (null to clear)     ║
║  - /skus/{id}/regional_prices/{id}/stock                               ║
║  - /skus/{id}/regional_prices/{id}/is_active                           ║
║                                                                        ║
║  Setting discount_price to null removes the discount:                  ║
║  { op: "replace",                                                      ║
║    path: "/skus/{skuId}/regional_prices/{id}/discount_price",          ║
║    value: null }                                                       ║
║                                                                        ║
║  Client-side validation:                                               ║
║  - original_price >= 0.01 and <= 9999999999.99                         ║
║  - discount_price (if set) >= 0.01, <= 9999999999.99,                  ║
║    and <= original_price                                               ║
║  - stock >= 0 (integer)                                                ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Price Cell Edit States

```
╔════════════════════════════════════════════════════════════════════════╗
║  REGIONAL PRICE CELL INTERACTIONS                                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  PRICE CELL lifecycle:                                                 ║
║                                                                        ║
║  DEFAULT            HOVER              EDIT                            ║
║  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                ║
║  │BDT 2,499.00  │  │BDT 2,499.00  │  │BDT [2499.00] │                ║
║  │(formatted)   │  │(cursor: text)│  │(blue border,  │                ║
║  └──────────────┘  └──────────────┘  │ raw number,   │                ║
║                                       │ currency label│                ║
║                                       │ outside)      │                ║
║                                       └──────────────┘                ║
║                                                                        ║
║  INVALID (discount > original):                                        ║
║  ┌──────────────┐                                                     ║
║  │BDT [3000.00] │  (red border)                                       ║
║  └──────────────┘                                                     ║
║  Discount cannot exceed original price                                 ║
║                                                                        ║
║  SAVED (brief feedback):                                               ║
║  ┌──────────────┐                                                     ║
║  │BDT 2,599.00 *│  (green * flash, 500ms, then returns to DEFAULT)    ║
║  └──────────────┘                                                     ║
║                                                                        ║
║  STOCK CELL with zero state:                                           ║
║                                                                        ║
║  NORMAL             ZERO STOCK                                         ║
║  ┌──────────────┐  ┌──────────────┐                                   ║
║  │  50           │  │  0           │                                   ║
║  │               │  │  Out of      │                                   ║
║  │               │  │  stock (BD)  │                                   ║
║  └──────────────┘  └──────────────┘                                   ║
║                     (red text, stop icon, region code in parentheses)  ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Toggle Regional Availability

```
╔════════════════════════════════════════════════════════════════════════╗
║  User toggles AE region to inactive                                   ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌────────┬──────────────┬──────────────┬───────┬────────┐           ║
║  │ AE     │ AED 149.00   │ ____________ │ 25    │ [off]  │           ║
║  │        │ (grayed out) │              │       │        │           ║
║  └────────┴──────────────┴──────────────┴───────┴────────┘           ║
║                                                                        ║
║  When is_active=false:                                                 ║
║  - Row appears grayed out                                              ║
║  - SKU is not purchasable in this region                               ║
║  - Price and stock fields remain editable                              ║
║                                                                        ║
║  For edit mode:                                                        ║
║  { op: "replace",                                                      ║
║    path: "/skus/{skuId}/regional_prices/{id}/is_active",               ║
║    value: false }                                                      ║
║                                                                        ║
║  Note: SKU is purchasable in a region only when BOTH:                  ║
║  - skus.is_active = true (the SKU itself is active)                    ║
║  - sku_regions.is_active = true (the regional entry is active)         ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Remove Regional Price

```
╔════════════════════════════════════════════════════════════════════════╗
║  User clicks [x] on the IN regional price row                        ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Before:                                                               ║
║  ┌────────┬──────────────┬──────────────┬───────┬────────┐           ║
║  │ BD     │ BDT 2,499.00│ BDT 2,199.00│ 50    │ [on]   │           ║
║  │ IN     │ INR 2,099.00│ ____________ │ 30    │ [on] [x]│          ║
║  │ AE     │ AED 149.00  │ ____________ │ 25    │ [on] [x]│          ║
║  └────────┴──────────────┴──────────────┴───────┴────────┘           ║
║                                                                        ║
║  No confirmation dialog (simple remove).                               ║
║                                                                        ║
║  After:                                                                ║
║  ┌────────┬──────────────┬──────────────┬───────┬────────┐           ║
║  │ BD     │ BDT 2,499.00│ BDT 2,199.00│ 50    │ [on]   │           ║
║  │ AE     │ AED 149.00  │ ____________ │ 25    │ [on] [x]│          ║
║  └────────┴──────────────┴──────────────┴───────┴────────┘           ║
║                                                                        ║
║  [+ Add Region] now shows IN as available again.                       ║
║                                                                        ║
║  For edit mode:                                                        ║
║  { op: "remove",                                                       ║
║    path: "/skus/{skuId}/regional_prices/{regionPriceId}" }             ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Bulk Copy Regional Pricing

```
╔════════════════════════════════════════════════════════════════════════╗
║  User clicks [Copy BD pricing to all SKUs]                            ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Source: SKU "Black/S" has BD regional price:                          ║
║  { region_id: 1, original_price: 2499.00, stock: 50, is_active: true }║
║                                                                        ║
║  ╔══════════════════════════════════════════════════════════════╗     ║
║  ║  Copy BD Pricing to All SKUs?                      [x]      ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                                              ║     ║
║  ║  Copy the BD regional pricing from "Black / S" to all        ║     ║
║  ║  other SKUs?                                                 ║     ║
║  ║                                                              ║     ║
║  ║  Price: BDT 2,499.00                                        ║     ║
║  ║  Stock: 50                                                   ║     ║
║  ║                                                              ║     ║
║  ║  This will:                                                  ║     ║
║  ║  - Add BD pricing to 5 SKUs that don't have it              ║     ║
║  ║  - Overwrite BD pricing on 3 SKUs that already have it      ║     ║
║  ║                                                              ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║              [Cancel]          [Copy Pricing]               ║     ║
║  ╚══════════════════════════════════════════════════════════════╝     ║
║                                                                        ║
║  For edit mode, the frontend generates patch ops per target SKU:        ║
║                                                                        ║
║  SKU already has BD pricing → replace ops:                             ║
║    { op: "replace",                                                    ║
║      path: "/skus/{id}/regional_prices/{rpId}/original_price",         ║
║      value: 2499.00 }                                                  ║
║    { op: "replace",                                                    ║
║      path: "/skus/{id}/regional_prices/{rpId}/discount_price",         ║
║      value: null }     (or source discount if set)                     ║
║    { op: "replace",                                                    ║
║      path: "/skus/{id}/regional_prices/{rpId}/stock",                  ║
║      value: 50 }                                                       ║
║                                                                        ║
║  SKU does NOT have BD pricing → add op:                                ║
║    { op: "add",                                                        ║
║      path: "/skus/{id}/regional_prices/-",                             ║
║      value: { region_id: 1, original_price: 2499.00,                  ║
║               discount_price: null, stock: 50, is_active: true } }     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Bulk Copy Feedback States

```
╔════════════════════════════════════════════════════════════════════════╗
║  BULK COPY BUTTON LIFECYCLE                                           ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  DEFAULT:                                                              ║
║  [Copy BD pricing to all SKUs]                                         ║
║  (blue text, outline button)                                           ║
║                                                                        ║
║  HOVER:                                                                ║
║  [Copy BD pricing to all SKUs]                                         ║
║  (darker blue, blue-50 bg)                                             ║
║                                                                        ║
║  NO SOURCE (BD has no pricing configured):                             ║
║  [Copy BD pricing to all SKUs]                                         ║
║  (gray text, disabled, cursor not-allowed)                             ║
║  Tooltip: "Configure BD pricing on at least one SKU first"             ║
║                                                                        ║
║  IN PROGRESS:                                                          ║
║  [Copying to 8 SKUs...]                                                ║
║  (spinner, disabled, blue text)                                        ║
║                                                                        ║
║  SUCCESS:                                                              ║
║  ┌──────────────────────────────────────────────┐                     ║
║  │ Copied BD pricing to 8 SKUs                   │  <- toast (green)   ║
║  └──────────────────────────────────────────────┘                     ║
║  (button resets to default state)                                      ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Performance Considerations

```
╔════════════════════════════════════════════════════════════════════════╗
║  PERFORMANCE STRATEGY FOR LARGE DATASETS                              ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Worst case: 100 SKUs x 6 regions = 600 price cells.                  ║
║  Rendering all at once causes layout thrashing and input lag.          ║
║                                                                        ║
║  Strategy: Accordion collapse + virtual scrolling                      ║
║                                                                        ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │  ▶ Black / S  — $24.99     3 regions            (collapsed)     │ ║
║  │  ▶ Black / M  — $24.99     3 regions            (collapsed)     │ ║
║  │  ▼ Black / L  — $26.99     3 regions            (expanded)      │ ║
║  │    ┌────────┬──────────┬──────────┬───────┬────────┐            │ ║
║  │    │ BD     │ BDT 2,599│ BDT 2,299│ 50    │ [on]   │            │ ║
║  │    │ IN     │ INR 2,099│ ________ │ 30    │ [on]   │            │ ║
║  │    │ AE     │ AED 149  │ ________ │ 25    │ [on]   │            │ ║
║  │    └────────┴──────────┴──────────┴───────┴────────┘            │ ║
║  │  ▶ White / S — $24.99     2 regions             (collapsed)     │ ║
║  │  ...                                                             │ ║
║  │  ▶ Red / L   — $26.99     0 regions             (collapsed)     │ ║
║  │                                                                  │ ║
║  │  ░░░░░░░░░ (virtual scroll — rows below viewport not rendered)  │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║                                                                        ║
║  Implementation:                                                       ║
║  - Default: all accordions collapsed (only headers rendered).         ║
║  - Only 1-2 SKUs expanded at a time (clicking a new one collapses     ║
║    the previous — optional UX choice, configurable).                  ║
║  - Accordion headers virtualized with @tanstack/react-virtual:        ║
║    only visible headers are in the DOM.                               ║
║  - Regional price data for a SKU loaded lazily on expand (if using    ║
║    expand[]=regional_prices, data is already in memory — just not     ║
║    rendered until the accordion opens).                                ║
║  - Quick fill and bulk copy operate on all SKUs regardless of         ║
║    accordion state.                                                    ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Currency Formatting

```
╔════════════════════════════════════════════════════════════════════════╗
║  CURRENCY DISPLAY RULES                                               ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Currency symbol/code derived from region.currency_code                ║
║  (from Core API regions endpoint, cached with TanStack Query).         ║
║                                                                        ║
║  Formatting via Intl.NumberFormat:                                      ║
║  Intl.NumberFormat(locale, {                                           ║
║    style: 'currency',                                                  ║
║    currency: region.currency_code                                      ║
║  }).format(parseFloat(price))                                          ║
║                                                                        ║
║  Examples:                                                             ║
║  BD: BDT 2,499.00     (Bangladeshi Taka)                              ║
║  IN: INR 2,099.00     (Indian Rupee, or ₹2,099.00)                   ║
║  AE: AED 149.00       (UAE Dirham)                                     ║
║  US: $24.99           (US Dollar)                                      ║
║  EU: €21.99           (Euro)                                           ║
║  AU: A$34.99          (Australian Dollar)                              ║
║                                                                        ║
║  Discount percentage (when discount_price is set):                     ║
║  ┌──────────────┬──────────────┐                                     ║
║  │ BDT 2,499.00 │ BDT 2,199.00│                                     ║
║  │              │ (12% off)    │  ← gray text, computed:              ║
║  └──────────────┴──────────────┘    ((original - discount) / original ║
║                                      * 100), rounded to nearest int   ║
║                                                                        ║
║  All price amounts right-aligned in table cells.                       ║
║  Input fields show raw numbers (no currency symbol while editing).     ║
║  Currency symbol/code appears as prefix or suffix depending on locale. ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Create Mode Data Shape

```
In the POST /seller-products body, regional prices nest under each SKU:

{
  "skus": [
    {
      "price": 24.99,
      "stock": 100,
      "sku_code": "BK-S",
      "is_active": true,
      "property_values": [[0, 0], [1, 0]],
      "regional_prices": [
        {
          "region_id": 1,
          "original_price": 2499.00,
          "discount_price": 2199.00,
          "stock": 50,
          "is_active": true
        },
        {
          "region_id": 2,
          "original_price": 2099.00,
          "discount_price": null,
          "stock": 30,
          "is_active": true
        }
      ]
    }
  ]
}
```

---

## Edit Mode Patch Paths

```
All regional pricing patch operations (see 03-product-edit-flow.md for full path table):

Add regional price to a SKU:
  op: "add"
  path: /skus/{skuId}/regional_prices/-
  value: {
    region_id:       integer (required, must exist in Core regions table)
    original_price:  number  (required, min 0.01, max 9999999999.99)
    discount_price:  number|null (optional, min 0.01 when set, <= original_price)
    stock:           integer (required, min 0)
    is_active:       boolean (optional, default true)
  }

Remove regional price from a SKU:
  op: "remove"
  path: /skus/{skuId}/regional_prices/{regionPriceId}

Replace individual fields:
  /skus/{skuId}/regional_prices/{id}/original_price  number (min 0.01)
  /skus/{skuId}/regional_prices/{id}/discount_price  number (min 0.01) or null
  /skus/{skuId}/regional_prices/{id}/stock           integer (min 0)
  /skus/{skuId}/regional_prices/{id}/is_active       boolean

Test (verify before mutating):
  op: "test" is valid on all paths above.
  Aborts entire patch with 409 if value does not match.
```

---

## Validation Summary

```
Client-side validation (prevent submission):
┌─────────────────────────────────────────┬──────────────────────────────────────┐
│ Rule                                    │ Client Error                         │
├─────────────────────────────────────────┼──────────────────────────────────────┤
│ original_price missing                  │ "Price is required"                  │
│ original_price < 0.01                   │ "Price must be at least 0.01"        │
│ original_price > 9999999999.99          │ "Price exceeds maximum"              │
│ discount_price > original_price         │ "Discount cannot exceed price"       │
│ discount_price < 0.01 (when set)        │ "Discount must be at least 0.01"    │
│ stock missing                           │ "Stock is required"                  │
│ stock < 0                               │ "Stock cannot be negative"           │
│ Duplicate region within SKU             │ "Region already configured"          │
└─────────────────────────────────────────┴──────────────────────────────────────┘

Server error messages (422 — use for mapServerErrors):
┌─────────────────────────────────────────────────────────────────────────┐
│ Create (from AgentSellerProductStoreRequest):                           │
│  region_id field: "exists:regions,id" → invalid returns standard 422   │
│                                                                         │
│ PATCH (from JsonPatchProcessor):                                        │
│  "Regional price must include region_id, original_price, and stock."    │
│  "Region ID {id} does not exist."                                       │
│  "Region ID {id} is already assigned to this SKU."                      │
│  "Regional original price must be at least 0.01."                       │
│  "Regional discount price must be at least 0.01 or null."              │
│  "Regional stock must be a non-negative integer."                       │
│                                                                         │
│ Note: discount_price exceeding original_price is a post-operation       │
│ constraint — validated after all patch ops complete.                     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Loading Regional Price Data (Edit Mode)

```
When loading a product for edit with expand[]=regional_prices:

GET /seller-products/{id}?expand[]=regional_prices

Note: expand[]=regional_prices implicitly includes expand[]=skus.
No need to pass both — regional_prices alone loads SKUs with their regional prices.

To load ALL regions for all SKUs, do NOT pass region_id.
Passing region_id would filter to a single region per SKU (useful for list page, not edit).

Each SKU in the response includes regional_prices:

{
  "object": "SellerProductSku",
  "id": 301,
  "price": "24.99",
  "stock": 100,
  "sku_code": "BK-S",
  "is_active": true,
  "associations": { "data": [...] },
  "regional_prices": {
    "data": [
      {
        "object": "SellerProductSkuRegionPrice",
        "id": 501,
        "region_id": 1,
        "original_price": "2499.00",
        "discount_price": "2199.00",
        "stock": 50,
        "is_active": true
      },
      {
        "object": "SellerProductSkuRegionPrice",
        "id": 502,
        "region_id": 2,
        "original_price": "2099.00",
        "discount_price": null,
        "stock": 30,
        "is_active": true
      }
    ]
  }
}

Field types from API:
  id:              BIGINT (integer)
  region_id:       BIGINT (integer) — FK to Core API regions table
  original_price:  string (DECIMAL(12,2) serialized as string like "2499.00")
  discount_price:  string or null
  stock:           integer
  is_active:       boolean

Regional price IDs (501, 502) are used in PATCH paths for updates.
region_id maps to a Core API region. Frontend must look up region name,
code, and currency from the cached region list.
```

---

## Empty States

```
╔════════════════════════════════════════════════════════════════════════╗
║  REGIONAL PRICING EMPTY STATES                                        ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  No regional prices configured for a SKU (collapsed accordion):        ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │  ▶ Black / S  — $24.99     No regions configured  [+ Add Region]│ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║  (gray italic "No regions configured" text, inline [+ Add Region])   ║
║                                                                        ║
║  All regions already added (dropdown disabled):                        ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │  [+ Add Region ▼]                                                │ ║
║  │  ┌────────────────────────────────────────────────────────────┐  │ ║
║  │  │  All available regions configured ✓                        │  │ ║
║  │  │  (disabled state, green check)                             │  │ ║
║  │  └────────────────────────────────────────────────────────────┘  │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Accessibility

```
╔════════════════════════════════════════════════════════════════════════╗
║  ACCESSIBILITY REQUIREMENTS                                           ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Accordion keyboard support:                                           ║
║  - Enter / Space: toggle expand/collapse on focused accordion header  ║
║  - Arrow Up / Down: move focus between accordion headers              ║
║  - Home / End: move to first / last accordion header                  ║
║                                                                        ║
║  Price input labels:                                                   ║
║  - aria-label="Original price for {region} on SKU {sku_code}"        ║
║  - aria-label="Discount price for {region} on SKU {sku_code}"        ║
║  - aria-label="Stock for {region} on SKU {sku_code}"                 ║
║                                                                        ║
║  Toggle switch labels:                                                 ║
║  - aria-label="Enable {region} pricing for SKU {sku_code}"           ║
║  - role="switch", aria-checked="true|false"                           ║
║                                                                        ║
║  Screen reader announcements:                                          ║
║  - On bulk copy completion: aria-live="polite" region announces       ║
║    "Copied {region} pricing to {n} SKUs"                              ║
║  - On region add/remove: aria-live="polite" announces change          ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```
