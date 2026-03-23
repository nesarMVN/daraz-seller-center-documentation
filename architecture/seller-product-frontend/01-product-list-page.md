# Product List Page

## Overview

The product list page is the main hub for managing all seller products. It displays products in a paginated table with status tabs, search, category filtering, sorting, and bulk operations.

**Route:** `/[locale]/(seller)/products`

---

## Page Layout

```
╔═════════════════════════════════════════════════════════════════════════════╗
║  MoveOn Seller Center                              John Doe   [Bell] [Cog]  ║
╠═════════════════════════════════════════════════════════════════════════════╣
║                                                                             ║
║  Products > Manage Products                                                 ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  STATUS TABS                                                        │   ║
║  │  ┌─────┐ ┌──────┐ ┌────────┐ ┌───────┐ ┌───────┐ ┌─────────┐ ┌──┐│   ║
║  │  │ All │ │Active│ │Inactive│ │ Draft │ │Pending│ │Violation│ │De││   ║
║  │  │(47) │ │ (23) │ │  (8)   │ │  (5)  │ │  (4)  │ │   (2)   │ │(5││   ║
║  │  └─────┘ └──────┘ └────────┘ └───────┘ └───────┘ └─────────┘ └──┘│   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  TOOLBAR                                                            │   ║
║  │  ┌──────────────────────────┐ ┌────────────────┐ ┌───────────────┐ │   ║
║  │  │ Search products...       │ │ All Categories │ │  + New Product│ │   ║
║  │  └──────────────────────────┘ └────────────────┘ └───────────────┘ │   ║
║  │                                                                     │   ║
║  │  Sort: [Created Date v]  [Desc v]        Region: [All Regions v]   │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  PRODUCT TABLE                                                      │   ║
║  │                                                                     │   ║
║  │  ┌──┬──────────────────────────────┬────────┬─────────┬──────────┐ │   ║
║  │  │[ ]│ Product                      │ Status │ Price   │ SKUs     │ │   ║
║  │  ├──┼──────────────────────────────┼────────┼─────────┼──────────┤ │   ║
║  │  │[ ]│ ┌─────┐ KingBank DDR5 RAM   │[Active]│ $265.99 │ 1 SKU    │ │   ║
║  │  │   │ │ IMG │ Electronics > RAM    │        │         │          │ │   ║
║  │  │   │ └─────┘                      │        │         │          │ │   ║
║  │  ├──┼──────────────────────────────┼────────┼─────────┼──────────┤ │   ║
║  │  │[ ]│ ┌─────┐ Cotton T-Shirt      │[Draft] │ $12-$18 │ 6 SKUs   │ │   ║
║  │  │   │ │ IMG │ Apparel > T-Shirts   │        │         │          │ │   ║
║  │  │   │ └─────┘                      │        │         │          │ │   ║
║  │  ├──┼──────────────────────────────┼────────┼─────────┼──────────┤ │   ║
║  │  │[ ]│ ┌─────┐ Wireless Mouse      │[Active]│ $24.99  │ 3 SKUs   │ │   ║
║  │  │   │ │ IMG │ Electronics > Mouse  │        │         │          │ │   ║
║  │  │   │ └─────┘                      │        │         │          │ │   ║
║  │  └──┴──────────────────────────────┴────────┴─────────┴──────────┘ │   ║
║  │                                                                     │   ║
║  │  ┌────────────────────────────────────────────────────────────────┐ │   ║
║  │  │  PAGINATION                                                    │ │   ║
║  │  │  < Prev   [1] [2] [3] ... [5]   Next >                        │ │   ║
║  │  │           Page 1 of 5 (47 products)     Per page: [20 v]       │ │   ║
║  │  └────────────────────────────────────────────────────────────────┘ │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ╔═════════════════════════════════════════════════════════════════════╗   ║
║  ║  BULK ACTION BAR (visible when items selected)                      ║   ║
║  ║  3 products selected  [Activate] [Deactivate] [Delete] [Cancel]    ║   ║
║  ╚═════════════════════════════════════════════════════════════════════╝   ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

---

## URL State Management

```
/products
  ?status=active                    ← Status tab filter
  &title=kingbank                   ← Search query
  &shipping_category_id=9ce8d...    ← Category filter (UUID)
  &sort_by=created_at               ← Sort field (title | created_at | updated_at | price)
  &sort_order=desc                  ← Sort direction (asc | desc)
  &page=1                           ← Current page
  &per_page=20                      ← Items per page (max 100)
  &region_id=3                      ← Regional pricing filter

Example: /products?status=active&title=ram&sort_by=price&sort_order=asc&page=2
```

All URL params map 1:1 to API query parameters. Changing any param triggers a new `GET /seller-products` call.

---

## User Workflows

### 1. Initial Page Load

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: User navigates to /products                                  ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Browser URL: /products                                                ║
║  (no query params = defaults: status=null, sort_by=created_at,         ║
║   sort_order=desc, page=1, per_page=20)                                ║
║                                                                        ║
║  Two API calls fire in parallel:                                       ║
║                                                                        ║
║  1. GET /seller-products/status-counts                                 ║
║     Response: {                                                        ║
║       data: {                                                          ║
║         all: 47, draft: 5, pending: 4, active: 23,                     ║
║         inactive: 8, violation: 2, deleted: 5                          ║
║       }                                                                ║
║     }                                                                  ║
║     Note: "all" = sum of ALL statuses including deleted.               ║
║     The "All" tab shows all non-deleted products by default            ║
║     (no status filter), so its row count may be less than the          ║
║     badge number. The badge reflects total inventory including         ║
║     deleted items.                                                     ║
║                                                                        ║
║  2. GET /seller-products?response_type=minimal&sort_by=created_at      ║
║     &sort_order=desc&page=1&per_page=20                                ║
║     Response: {                                                        ║
║       data: [ ...20 SellerProductMinimal objects ],                     ║
║       pagination: { current_page: 1, total: 47, last_page: 3 }        ║
║     }                                                                  ║
║                                                                        ║
║  Page renders:                                                         ║
║  ┌─────────────────────────────────────────────────────────────┐      ║
║  │  [All(47)] [Active(23)] [Inactive(8)] [Draft(5)]            │      ║
║  │  [Pending(4)] [Violation(2)] [Deleted(5)]                   │      ║
║  │                                                             │      ║
║  │  Search: [_______________]  Category: [All]                 │      ║
║  │                                                             │      ║
║  │  ┌──┬─────────────────────┬────────┬─────────┬────────┐    │      ║
║  │  │[ ]│ Product              │ Status │ Price   │ SKUs   │    │      ║
║  │  ├──┼─────────────────────┼────────┼─────────┼────────┤    │      ║
║  │  │[ ]│ KingBank DDR5 RAM   │ Active │ $265.99 │ 1      │    │      ║
║  │  │[ ]│ Cotton T-Shirt      │ Draft  │ $12-$18 │ 6      │    │      ║
║  │  │[ ]│ Wireless Mouse      │ Active │ $24.99  │ 3      │    │      ║
║  │  │   │ ...17 more rows     │        │         │        │    │      ║
║  │  └──┴─────────────────────┴────────┴─────────┴────────┘    │      ║
║  │                                                             │      ║
║  │  < Prev  [1] 2  3  Next >     Page 1 of 3 (47 total)       │      ║
║  └─────────────────────────────────────────────────────────────┘      ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### 2. Filter by Status Tab

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: User clicks "Active" tab                                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  URL updates: /products?status=active                                  ║
║  (page resets to 1, search/category cleared)                           ║
║                                                                        ║
║  API Call:                                                             ║
║  GET /seller-products?response_type=minimal&status=active               ║
║      &sort_by=created_at&sort_order=desc&page=1&per_page=20            ║
║                                                                        ║
║  ┌─────────────────────────────────────────────────────────────┐      ║
║  │  [All(47)] [Active(23)] [Inactive(8)] [Draft(5)]            │      ║
║  │            ^^^^^^^^^^^^                                      │      ║
║  │            Selected                                          │      ║
║  │                                                             │      ║
║  │  ┌──┬─────────────────────┬────────┬─────────┬────────┐    │      ║
║  │  │[ ]│ Product              │ Status │ Price   │ SKUs   │    │      ║
║  │  ├──┼─────────────────────┼────────┼─────────┼────────┤    │      ║
║  │  │[ ]│ KingBank DDR5 RAM   │ Active │ $265.99 │ 1      │    │      ║
║  │  │[ ]│ Wireless Mouse      │ Active │ $24.99  │ 3      │    │      ║
║  │  │[ ]│ USB-C Hub           │ Active │ $39.99  │ 2      │    │      ║
║  │  │   │ ...17 more rows     │        │         │        │    │      ║
║  │  └──┴─────────────────────┴────────┴─────────┴────────┘    │      ║
║  │                                                             │      ║
║  │  < Prev  [1] 2  Next >     Page 1 of 2 (23 total)          │      ║
║  └─────────────────────────────────────────────────────────────┘      ║
║                                                                        ║
║  What happened:                                                        ║
║  - "Active" tab highlighted                                            ║
║  - Table shows only active products                                    ║
║  - Pagination reflects 23 active products                              ║
║  - Status counts remain cached (no re-fetch)                           ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### 3. Search Products

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: User types "ram" in search box                               ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌──────────────────────────────────────────┐                         ║
║  │  Search: [ram______________] Searching...│                         ║
║  └──────────────────────────────────────────┘                         ║
║                                                                        ║
║  Wait 300ms debounce...                                                ║
║                                                                        ║
║  URL updates: /products?status=active&title=ram                        ║
║                                                                        ║
║  API Call:                                                             ║
║  GET /seller-products?response_type=minimal&status=active&title=ram     ║
║      &sort_by=created_at&sort_order=desc&page=1&per_page=20            ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
                                    │
                                    ▼
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 2: Results displayed                                            ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌─────────────────────────────────────────────────────────────┐      ║
║  │  Search: [ram______________] [x Clear]                      │      ║
║  │                                                             │      ║
║  │  ┌──┬─────────────────────┬────────┬─────────┬────────┐    │      ║
║  │  │[ ]│ Product              │ Status │ Price   │ SKUs   │    │      ║
║  │  ├──┼─────────────────────┼────────┼─────────┼────────┤    │      ║
║  │  │[ ]│ KingBank DDR5 RAM   │ Active │ $265.99 │ 1      │    │      ║
║  │  │[ ]│ Corsair DDR4 RAM    │ Active │ $89.99  │ 4      │    │      ║
║  │  └──┴─────────────────────┴────────┴─────────┴────────┘    │      ║
║  │                                                             │      ║
║  │  2 results found                                            │      ║
║  └─────────────────────────────────────────────────────────────┘      ║
║                                                                        ║
║  Clicking [x Clear] removes &title= from URL, resets to full list.    ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Search Input Interaction States

```
╔════════════════════════════════════════════════════════════════════════╗
║  SEARCH INPUT STATES                                                  ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  1. DEFAULT:                                                           ║
║  ┌──────────────────────────────────────────┐                         ║
║  │  🔍 Search products...                    │                         ║
║  └──────────────────────────────────────────┘                         ║
║  (gray border, placeholder text, search icon left)                     ║
║                                                                        ║
║  2. FOCUSED:                                                           ║
║  ┌──────────────────────────────────────────┐                         ║
║  │  🔍 |                                     │                         ║
║  └──────────────────────────────────────────┘                         ║
║  (blue border, cursor active, placeholder hidden)                      ║
║                                                                        ║
║  3. TYPING (debounce active):                                          ║
║  ┌──────────────────────────────────────────┐                         ║
║  │  🔍 ram                              ⏳   │                         ║
║  └──────────────────────────────────────────┘                         ║
║  (⏳ debounce indicator, 300ms wait before search fires)               ║
║                                                                        ║
║  4. POPULATED (results loaded):                                        ║
║  ┌──────────────────────────────────────────┐                         ║
║  │  🔍 ram                              [x]  │                         ║
║  └──────────────────────────────────────────┘                         ║
║  (value shown, [x] clear button visible on right)                      ║
║                                                                        ║
║  5. ERROR (search API failed):                                         ║
║  ┌──────────────────────────────────────────┐                         ║
║  │  🔍 ram                              [x]  │                         ║
║  └──────────────────────────────────────────┘                         ║
║  ⚠️ Search failed. [Retry]                                            ║
║  (red border, error message + retry link below)                        ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### 4. Sort Products

```
╔════════════════════════════════════════════════════════════════════════╗
║  User changes sort to "Price" ascending                               ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Before:                                                               ║
║  Sort: [Created Date v] [Desc v]                                       ║
║                                                                        ║
║  User clicks [Created Date v] -> selects "Price"                       ║
║  User clicks [Desc v] -> selects "Asc"                                 ║
║                                                                        ║
║  URL updates: /products?status=active&sort_by=price&sort_order=asc     ║
║                                                                        ║
║  API Call:                                                             ║
║  GET /seller-products?response_type=minimal&status=active               ║
║      &sort_by=price&sort_order=asc&page=1&per_page=20                  ║
║                                                                        ║
║  After:                                                                ║
║  Sort: [Price v] [Asc v]                                               ║
║                                                                        ║
║  ┌──┬─────────────────────┬────────┬──────────┬────────┐              ║
║  │[ ]│ Product              │ Status │ Price  v │ SKUs   │              ║
║  ├──┼─────────────────────┼────────┼──────────┼────────┤              ║
║  │[ ]│ Phone Case           │ Active │ $4.99    │ 8      │              ║
║  │[ ]│ USB Cable            │ Active │ $7.99    │ 2      │              ║
║  │[ ]│ Cotton T-Shirt       │ Active │ $12.00   │ 6      │              ║
║  └──┴─────────────────────┴────────┴──────────┴────────┘              ║
║                                                                        ║
║  Note: sort_by=price sorts by denormalized min_price column.           ║
║  When region_id is active, sorts by regional min price instead.        ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### 5. Filter by Category

```
╔════════════════════════════════════════════════════════════════════════╗
║  User opens category dropdown and selects a leaf category              ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌─────────────────────────────────────────────────────────────┐      ║
║  │  Category: [All Categories v]                                │      ║
║  │                                                             │      ║
║  │  User clicks dropdown:                                      │      ║
║  │  ┌───────────────────────────────────────┐                  │      ║
║  │  │  Search categories...                  │                  │      ║
║  │  ├───────────────────────────────────────┤                  │      ║
║  │  │  All Categories                        │                  │      ║
║  │  │  v Electric Equipment & Telecom        │                  │      ║
║  │  │    v Solar Energy Products             │                  │      ║
║  │  │      - Calculator                      │                  │      ║
║  │  │      - Solar Panel                     │                  │      ║
║  │  │    v Computer Peripherals              │                  │      ║
║  │  │      - Keyboard                        │                  │      ║
║  │  │      - Mouse   <-- clicks here         │                  │      ║
║  │  │  v Agriculture & Food                  │                  │      ║
║  │  │    ...                                 │                  │      ║
║  │  └───────────────────────────────────────┘                  │      ║
║  └─────────────────────────────────────────────────────────────┘      ║
║                                                                        ║
║  URL updates:                                                          ║
║  /products?status=active&shipping_category_id=9ce8d85f-xxxx            ║
║                                                                        ║
║  API Call:                                                             ║
║  GET /seller-products?response_type=minimal&status=active               ║
║      &shipping_category_id=9ce8d85f-xxxx&sort_by=created_at            ║
║      &sort_order=desc&page=1&per_page=20                               ║
║                                                                        ║
║  Category dropdown now shows:                                          ║
║  [Electric Equipment > Computer Peripherals > Mouse  x]                ║
║                                                                        ║
║  The [x] clears the category filter.                                   ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

The category tree is fetched from the shipping category API:
- `GET /api/shipping-core/customer/shipping-category/v1/shipping-categories`
- Returns 18 top-level categories with nested children (3 levels max)
- Cache with TanStack Query (staleTime: 30 minutes -- categories change rarely)

### 6. Regional Pricing Filter

```
╔════════════════════════════════════════════════════════════════════════╗
║  User selects a region to view regional prices                        ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌─────────────────────────────────────────────────────────────┐      ║
║  │  Region: [All Regions v]                                    │      ║
║  │                                                             │      ║
║  │  User clicks dropdown:                                      │      ║
║  │  ┌─────────────────────────┐                                │      ║
║  │  │  All Regions            │                                │      ║
║  │  │  Bangladesh (BD)        │                                │      ║
║  │  │  India (IN)             │                                │      ║
║  │  │  UAE (AE)       <--     │                                │      ║
║  │  │  USA (US)               │                                │      ║
║  │  │  EU (EU)                │                                │      ║
║  │  │  Australia (AU)         │                                │      ║
║  │  └─────────────────────────┘                                │      ║
║  └─────────────────────────────────────────────────────────────┘      ║
║                                                                        ║
║  URL updates: /products?region_id=3                                    ║
║                                                                        ║
║  API Call:                                                             ║
║  GET /seller-products?response_type=minimal&region_id=3                 ║
║      &sort_by=created_at&sort_order=desc&page=1&per_page=20            ║
║                                                                        ║
║  Behavior changes:                                                     ║
║  - Only products with active SKUs in UAE region shown                  ║
║    (requires BOTH sku.is_active=true AND sku_region.is_active=true)   ║
║  - Price column shows regional original_price range only               ║
║    (discount_price is NOT reflected in list price_range)               ║
║  - Products without UAE pricing are excluded entirely                  ║
║  - Sort by price uses regional min original_price instead of base      ║
║                                                                        ║
║  ┌──┬─────────────────────┬────────┬──────────────┬────────┐          ║
║  │[ ]│ Product              │ Status │ Price (AE)   │ SKUs   │          ║
║  ├──┼─────────────────────┼────────┼──────────────┼────────┤          ║
║  │[ ]│ KingBank DDR5 RAM   │ Active │ AED 975.00   │ 1      │          ║
║  │[ ]│ USB-C Hub           │ Active │ AED 146.00   │ 2      │          ║
║  └──┴─────────────────────┴────────┴──────────────┴────────┘          ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

The region list is fetched from the Core module API:
- `GET /api/core/regions` (or equivalent Core endpoint)
- Returns regions with: id, name, code, currency_id, status
- 6 supported regions: BD, IN, AE, US, EU, AU
- Cache with TanStack Query (staleTime: 30 minutes -- regions change rarely)
- Query key: `["regions"]`
- Currency is derived from `region.currency_id` at display time (not stored on pricing rows)

---

### 7. Bulk Operations

#### Select Products

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: User selects multiple products                               ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌──┬─────────────────────┬────────┬─────────┬────────┐              ║
║  │[x]│ Product              │ Status │ Price   │ SKUs   │              ║
║  ├──┼─────────────────────┼────────┼─────────┼────────┤              ║
║  │[x]│ KingBank DDR5 RAM   │ Active │ $265.99 │ 1      │              ║
║  │[ ]│ Cotton T-Shirt      │ Active │ $12-$18 │ 6      │              ║
║  │[x]│ Wireless Mouse      │ Active │ $24.99  │ 3      │              ║
║  │[x]│ USB-C Hub           │ Active │ $39.99  │ 2      │              ║
║  └──┴─────────────────────┴────────┴─────────┴────────┘              ║
║                                                                        ║
║  Header checkbox: Click to select/deselect all on current page.        ║
║  Individual checkboxes: Toggle individual products.                    ║
║  Selection is current-page only (no cross-page selection).             ║
║                                                                        ║
║  API limit: Bulk operations accept max 50 product IDs per request.     ║
║  Since per_page max is 100, selection can exceed the API limit.        ║
║                                                                        ║
║  When selection > 50, the frontend batches into multiple API calls:    ║
║  - Split selected IDs into chunks of 50                                ║
║  - Execute sequentially (not parallel, to avoid race conditions)       ║
║  - Aggregate results: sum updated_count, merge skipped arrays          ║
║  - If any batch returns 403, stop and show error for remaining         ║
║                                                                        ║
║  When 1+ products selected, bulk action bar appears at bottom:         ║
║                                                                        ║
║  ╔═════════════════════════════════════════════════════════════════╗   ║
║  ║  3 products selected                                            ║   ║
║  ║  [Activate] [Deactivate] [Delete]           [Cancel Selection] ║   ║
║  ╚═════════════════════════════════════════════════════════════════╝   ║
║                                                                        ║
║  Available bulk actions depend on current tab:                         ║
║  - Active tab: [Deactivate] [Delete]                                   ║
║  - Inactive tab: [Activate] [Delete]                                   ║
║  - Draft tab: [Delete]                                                 ║
║  - All tab: [Activate] [Deactivate] [Delete]                           ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

#### Bulk Deactivate

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 2: User clicks [Deactivate]                                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ╔══════════════════════════════════════════════════╗                  ║
║  ║  Deactivate 3 Products?              [x]        ║                  ║
║  ╠══════════════════════════════════════════════════╣                  ║
║  ║                                                  ║                  ║
║  ║  The following products will be deactivated:     ║                  ║
║  ║                                                  ║                  ║
║  ║  - KingBank DDR5 RAM                             ║                  ║
║  ║  - Wireless Mouse                                ║                  ║
║  ║  - USB-C Hub                                     ║                  ║
║  ║                                                  ║                  ║
║  ║  Deactivated products will not be visible to     ║                  ║
║  ║  buyers until reactivated.                       ║                  ║
║  ║                                                  ║                  ║
║  ╠══════════════════════════════════════════════════╣                  ║
║  ║                   [Cancel]  [Deactivate]        ║                  ║
║  ╚══════════════════════════════════════════════════╝                  ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
                                    │
                          User clicks [Deactivate]
                                    │
                                    ▼
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 3: API call and response                                        ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  API Call:                                                             ║
║  POST /seller-products/bulk-status                                     ║
║  Body: {                                                               ║
║    product_ids: [1, 3, 4],                                             ║
║    status: "inactive"                                                  ║
║  }                                                                     ║
║                                                                        ║
║  Response (200):                                                       ║
║  {                                                                     ║
║    message: "Bulk status update completed.",                          ║
║    updated_count: 3,                                                   ║
║    skipped_count: 0,                                                   ║
║    skipped: []                                                         ║
║  }                                                                     ║
║                                                                        ║
║  After success:                                                        ║
║  - Toast: "3 products deactivated"                                     ║
║  - Invalidate product list query cache                                 ║
║  - Invalidate status counts query cache                                ║
║  - Clear selection state                                               ║
║  - Products disappear from Active tab (or update status badge)         ║
║                                                                        ║
║  Partial success (some skipped):                                       ║
║  Response: {                                                           ║
║    updated_count: 2,                                                   ║
║    skipped_count: 1,                                                   ║
║    skipped: [{ id: 4, reason: "Cannot transition from draft" }]        ║
║  }                                                                     ║
║  Toast: "2 products deactivated. 1 skipped (invalid status)."          ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

#### Bulk Delete

```
╔════════════════════════════════════════════════════════════════════════╗
║  User clicks [Delete] with 3 products selected                        ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ╔══════════════════════════════════════════════════╗                  ║
║  ║  Delete 3 Products?                  [x]        ║                  ║
║  ╠══════════════════════════════════════════════════╣                  ║
║  ║                                                  ║                  ║
║  ║  The following products will be deleted:          ║                  ║
║  ║                                                  ║                  ║
║  ║  - KingBank DDR5 RAM                             ║                  ║
║  ║  - Wireless Mouse                                ║                  ║
║  ║  - USB-C Hub                                     ║                  ║
║  ║                                                  ║                  ║
║  ║  ┌──────────────────────────────────────────┐   ║                  ║
║  ║  │  Deleted products can be viewed in the   │   ║                  ║
║  ║  │  "Deleted" tab but cannot be restored.   │   ║                  ║
║  ║  └──────────────────────────────────────────┘   ║                  ║
║  ║                                                  ║                  ║
║  ╠══════════════════════════════════════════════════╣                  ║
║  ║              [Cancel]  [Delete Products]        ║                  ║
║  ╚══════════════════════════════════════════════════╝                  ║
║                                                                        ║
║  API Call:                                                             ║
║  POST /seller-products/bulk-delete                                     ║
║  Body: { product_ids: [1, 3, 4] }                                     ║
║                                                                        ║
║  Response (200):                                                       ║
║  { message: "Products deleted successfully.", deleted_count: 3 }       ║
║                                                                        ║
║  After success:                                                        ║
║  - Toast: "3 products deleted"                                         ║
║  - Invalidate product list + status counts                             ║
║  - Clear selection                                                     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

> Future: Bulk import/export via CSV is planned but not in MVP scope.

> Future: Product analytics link (views, conversion rate) in table row — not in MVP.

### 8. Navigate to Product Detail

```
╔════════════════════════════════════════════════════════════════════════╗
║  User clicks a product row                                            ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌──┬─────────────────────┬────────┬─────────┬────────┐              ║
║  │[ ]│ KingBank DDR5 RAM   │ Active │ $265.99 │ 1      │              ║
║  │   │ ^^^^^^^^^^^^^^^^^^^^^^^                            │              ║
║  │   │ Clickable product name/row                         │              ║
║  └──┴─────────────────────┴────────┴─────────┴────────┘              ║
║                                    │                                   ║
║                                    │ Click on product name/row         ║
║                                    ▼                                   ║
║  Router navigates to: /products/{id}/edit                              ║
║                                                                        ║
║  Product row also has a context menu (three-dot icon):                 ║
║  ┌─────────────────────┐                                              ║
║  │  Edit               │  -> /products/{id}/edit                       ║
║  │  Duplicate          │  -> /products/create?source_product_id={id}   ║
║  │  ──────────────     │                                               ║
║  │  Activate           │  -> PATCH status                              ║
║  │  Deactivate         │  -> PATCH status                              ║
║  │  ──────────────     │                                               ║
║  │  Delete             │  -> DELETE with confirm                       ║
║  └─────────────────────┘                                              ║
║                                                                        ║
║  Available actions depend on current product status:                   ║
║  - draft:     [Edit] [Duplicate] [Delete]                              ║
║  - active:    [Edit] [Duplicate] [Deactivate] [Delete]                 ║
║  - inactive:  [Edit] [Duplicate] [Activate] [Delete]                   ║
║  - pending:   [Edit] [Duplicate]                                       ║
║  - violation: [Edit] [Duplicate]                                       ║
║    Violation row shows reason icon: ⚠️ tooltip with violation reason    ║
║    from API. Clicking navigates to edit page with violation banner.     ║
║    (See 03-product-edit-flow.md "Violation Resubmission Workflow".)    ║
║  - deleted:   (view only, no actions)                                  ║
║    Deleted rows show deleted_at date column. Rows are non-clickable    ║
║    (or click opens read-only view). No action buttons in context menu  ║
║    (only "View"). No bulk actions available on Deleted tab.            ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### 9. Single Product Delete (Context Menu)

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: User clicks [...] on a product row, selects "Delete"         ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ╔══════════════════════════════════════════════════╗                  ║
║  ║  Delete Product?                        [x]      ║                  ║
║  ╠══════════════════════════════════════════════════╣                  ║
║  ║                                                  ║                  ║
║  ║  Are you sure you want to delete this product?   ║                  ║
║  ║                                                  ║                  ║
║  ║  Product: KingBank DDR5 RAM                      ║                  ║
║  ║  Status: Active                                  ║                  ║
║  ║  SKUs: 6                                         ║                  ║
║  ║                                                  ║                  ║
║  ║  This product will be moved to the Deleted tab.  ║                  ║
║  ║                                                  ║                  ║
║  ╠══════════════════════════════════════════════════╣                  ║
║  ║              [Cancel]     [Delete Product]       ║                  ║
║  ╚══════════════════════════════════════════════════╝                  ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
                                    │
                        User clicks [Delete Product]
                                    │
                                    ▼
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 2: API call and response                                        ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  API Call:                                                             ║
║  DELETE /seller-products/{id}       ║
║                                                                        ║
║  Response (200):                                                       ║
║  { "message": "Product deleted successfully." }                       ║
║                                                                        ║
║  After success:                                                        ║
║  - Toast: "Product deleted"                                            ║
║  - Invalidate product list + status counts                             ║
║  - Product moves to Deleted tab (soft delete)                          ║
║                                                                        ║
║  Error (404): Product not found or unauthorized                        ║
║  - Toast: "Product not found"                                          ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### 10. Single Status Change (Context Menu)

```
╔════════════════════════════════════════════════════════════════════════╗
║  User clicks [...] -> "Deactivate" on an active product               ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Uses the same bulk-status endpoint with a single product ID:          ║
║                                                                        ║
║  API Call:                                                             ║
║  POST /seller-products/bulk-status                                     ║
║  Body: { "product_ids": [42], "status": "inactive" }                   ║
║                                                                        ║
║  Response (200):                                                       ║
║  { "updated_count": 1, "skipped_count": 0, "skipped": [] }            ║
║                                                                        ║
║  After success:                                                        ║
║  - Toast: "Product deactivated"                                        ║
║  - Invalidate product list + status counts                             ║
║  - Product disappears from Active tab (or badge updates)               ║
║                                                                        ║
║  Same for Activate: status="active"                                    ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### 11. Duplicate Product (Context Menu)

```
╔════════════════════════════════════════════════════════════════════════╗
║  User clicks [...] on a product row, selects "Duplicate"             ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Action: Navigate to the create page with the current product          ║
║  as a source for pre-filling.                                          ║
║                                                                        ║
║  Router navigates to:                                                  ║
║  /products/create?source_product_id={id}                               ║
║                                                                        ║
║  The create page detects the source_product_id param and:              ║
║  1. Fetches the existing product (GET /seller-products/{id})           ║
║  2. Pre-fills form fields from the response (see 02-product-create-    ║
║     flow.md, Section: Source Product Pre-fill)                         ║
║  3. Category must still be selected manually                           ║
║  4. Gallery images are previewed but the seller should re-upload       ║
║     from their own Media Center                                        ║
║  5. Submitting creates a NEW product (POST /seller-products)           ║
║                                                                        ║
║  The new product's source_product_id field links back to the           ║
║  original for tracking purposes.                                       ║
║                                                                        ║
║  Note: source_product_id is VARCHAR(30), matching the ULID format      ║
║  used by the vendor catalog product-service.                           ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Status Transition State Machine

```
                     ┌──────────────────────────────────────┐
                     │                                      │
                     │              ┌─────────┐             │
           ┌────────┼──────────────│  DRAFT  │             │
           │        │              └────┬────┘             │
           │        │                   │                   │
           │        │          Submit   │                   │
           │        │         for review│                   │
           │        │                   ▼                   │
           │        │              ┌─────────┐             │
           │        │   ┌──────────│ PENDING │──────┐      │
           │        │   │          └─────────┘      │      │
           │        │   │ Approve                Flag│      │
           │        │   │ (admin)            (admin) │      │
           │        │   ▼                            ▼      │
           │        │ ┌────────┐            ┌───────────┐  │
           │        │ │ ACTIVE │◄──────────►│ VIOLATION │  │
           │        │ └───┬────┘  Resubmit  └───────────┘  │
           │        │     │ ▲                               │
           │        │     │ │ Activate                      │
           │  Delete│     │ │                               │
           │  (any) │     │ Deactivate                      │
           │        │     ▼ │                               │
           │        │ ┌──────────┐                          │
           │        │ │ INACTIVE │                          │
           │        │ └──────────┘                          │
           │        │                                      │
           ▼        ▼                                      │
      ┌─────────┐                                          │
      │ DELETED │◄─────────────────────────────────────────┘
      │ (soft)  │  Any status can transition to deleted
      └─────────┘
```

Valid transitions for seller actions (frontend):

```
┌───────────┬──────────────────────────────────────┐
│ From      │ To (seller can trigger)              │
├───────────┼──────────────────────────────────────┤
│ draft     │ pending (submit for review)          │
│ active    │ inactive (deactivate)                │
│ inactive  │ active (activate)                    │
│ violation │ pending (resubmit after fix)         │
│ any       │ deleted (soft delete)                │
└───────────┴──────────────────────────────────────┘

Admin-only transitions (not available in seller UI):
┌───────────┬──────────────────────────────────────┐
│ pending   │ active (approve)                     │
│ pending   │ violation (flag)                     │
└───────────┴──────────────────────────────────────┘
```

---

## Loading States

```
┌─────────────────────────────────────────────────────────────────────┐
│  STATUS TABS (loading)                                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐              │
│  │ ░░░ (░░) │ │ ░░░ (░░) │ │ ░░░ (░░) │ │ ░░░ (░░) │              │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘              │
│                                                                     │
│  TABLE (loading - skeleton rows)                                    │
│  ┌──┬─────────────────────────────┬────────┬─────────┬────────┐    │
│  │░░│ ░░░░░░░░░░░░░░░░░░░░░░░░░░ │ ░░░░░░ │ ░░░░░░░ │ ░░░░░░│    │
│  │░░│ ░░░░░░░░░░░░░░░░░░░░░░░░░░ │ ░░░░░░ │ ░░░░░░░ │ ░░░░░░│    │
│  │░░│ ░░░░░░░░░░░░░░░░░░░░░░░░░░ │ ░░░░░░ │ ░░░░░░░ │ ░░░░░░│    │
│  │░░│ ░░░░░░░░░░░░░░░░░░░░░░░░░░ │ ░░░░░░ │ ░░░░░░░ │ ░░░░░░│    │
│  │░░│ ░░░░░░░░░░░░░░░░░░░░░░░░░░ │ ░░░░░░ │ ░░░░░░░ │ ░░░░░░│    │
│  └──┴─────────────────────────────┴────────┴─────────┴────────┘    │
│                                                                     │
│  Use shadcn Skeleton components for each cell.                      │
│  Show 5 skeleton rows matching the expected per_page layout.        │
└─────────────────────────────────────────────────────────────────────┘
```

### Detailed Loading Skeletons

```
╔════════════════════════════════════════════════════════════════════════╗
║  SKELETON ANATOMY — TABLE ROW                                         ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Each skeleton row matches the actual product row layout:              ║
║                                                                        ║
║  ┌──┬──────────────────────────────────────┬────────┬────────┬──────┐║
║  │░░│ ┌──────┐  ░░░░░░░░░░░░░░░░░░░░░░░░  │ ░░░░░░ │ ░░░░░░ │ ░░░░│║
║  │  │ │      │  ░░░░░░░░░░░░░░░            │        │        │      │║
║  │  │ │ ░░░░ │  ░░░░░░░░░░░                │        │        │      │║
║  │  │ └──────┘                              │        │        │      │║
║  └──┴──────────────────────────────────────┴────────┴────────┴──────┘║
║   ^^   ^^^^^^   ^^^^^^^^^^^^^^^^^^^^^^^^    ^^^^^^^^  ^^^^^^^^  ^^^^  ║
║   chk  img      title (random width)        status    price     SKUs  ║
║   box  rect     + category (shorter)        pill      bar       count ║
║                                                                        ║
║  Shimmer animation:                                                    ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  ░░░░░░▓▓▓░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │║
║  │         ^^^                                                      │║
║  │    shimmer highlight sweeps left-to-right, 1.5s loop             │║
║  └──────────────────────────────────────────────────────────────────┘║
║                                                                        ║
║  Title skeleton widths vary per row to look natural:                   ║
║  Row 1: ░░░░░░░░░░░░░░░░░░░░░░░░░░  (80%)                           ║
║  Row 2: ░░░░░░░░░░░░░░░░░░          (60%)                           ║
║  Row 3: ░░░░░░░░░░░░░░░░░░░░░░░░    (75%)                           ║
║  Row 4: ░░░░░░░░░░░░░░░░            (55%)                           ║
║  Row 5: ░░░░░░░░░░░░░░░░░░░░░░      (70%)                           ║
║                                                                        ║
║  BACKGROUND REFETCH state (data already loaded):                       ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  ═══════════════════════════════════════════════  ← thin progress│║
║  │  ┌──┬─────────────────────┬────────┬─────────┐  bar at top      │║
║  │  │[ ]│ KingBank DDR5 RAM   │ Active │ $265.99 │  (existing data  │║
║  │  │[ ]│ Cotton T-Shirt      │ Draft  │ $12-$18 │   at opacity 0.6)│║
║  │  │[ ]│ Wireless Mouse      │ Active │ $24.99  │                  │║
║  │  └──┴─────────────────────┴────────┴─────────┘                  │║
║  └──────────────────────────────────────────────────────────────────┘║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Empty States

### No Products at All (new seller)

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  [All(0)] [Active(0)] [Inactive(0)] [Draft(0)] ...                  │
│                                                                     │
│                          ┌─────────┐                                │
│                          │         │                                │
│                          │   Box   │                                │
│                          │         │                                │
│                          └─────────┘                                │
│                                                                     │
│              You have no products yet                                │
│                                                                     │
│         Start by creating your first product listing                │
│                                                                     │
│                    [+ Create Product]                                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### No Products in Current Filter

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  [All(47)] [Active(23)] [Inactive(8)] [Draft(5)] ...                │
│                        ^^^^^^^^^^                                   │
│  Search: [wireless keyboard]  Category: [All]                       │
│                                                                     │
│                          ┌─────────┐                                │
│                          │         │                                │
│                          │  Empty  │                                │
│                          │         │                                │
│                          └─────────┘                                │
│                                                                     │
│           No products match your search                             │
│                                                                     │
│       Try a different search term or clear filters                  │
│                                                                     │
│                   [Clear Filters]                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### No Products in Status Tab

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  [All(47)] ... [Violation(0)]                                       │
│                ^^^^^^^^^^^^^^                                       │
│                                                                     │
│                          ┌─────────┐                                │
│                          │         │                                │
│                          │  Check  │                                │
│                          │         │                                │
│                          └─────────┘                                │
│                                                                     │
│           No products with violation status                         │
│                                                                     │
│             All your products are in good standing                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### No Products in Region Filter

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Region: [Bangladesh ▼]                                             │
│                                                                     │
│                          ┌─────────┐                                │
│                          │         │                                │
│                          │  Globe  │                                │
│                          │         │                                │
│                          └─────────┘                                │
│                                                                     │
│           No products in Bangladesh                                 │
│                                                                     │
│       Products need regional pricing configured to appear here      │
│                                                                     │
│                   [Clear Filter]                                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Deleted Tab Empty State

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  [All(47)] ... [Deleted(0)]                                         │
│                                                                     │
│                          ┌─────────┐                                │
│                          │         │                                │
│                          │  Trash  │                                │
│                          │         │                                │
│                          └─────────┘                                │
│                                                                     │
│           No deleted products                                       │
│                                                                     │
│       Your product catalog is intact                                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Error States

### Network Error Loading Products

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                          ┌─────────┐                                │
│                          │         │                                │
│                          │  Error  │                                │
│                          │         │                                │
│                          └─────────┘                                │
│                                                                     │
│           Failed to load products                                   │
│                                                                     │
│         Check your connection and try again                         │
│                                                                     │
│                      [Retry]                                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Retry triggers TanStack Query refetch for the failed query.
```

### Invalid URL Parameter Error (422)

```
If the user manually edits the URL with invalid query params, the API returns 422.

Examples of invalid params:
- shipping_category_id=not-a-uuid  -> "The shipping_category_id must be a valid UUID."
- sort_by=popularity               -> "The sort_by must be one of: title, created_at, updated_at, price."
- sort_order=random                -> "The sort_order must be one of: asc, desc."
- per_page=500                     -> "The per_page must not be greater than 100."
- status=archived                  -> "The status must be one of: draft, pending, active, inactive, violation, deleted."
- region_id=abc                    -> "The region_id must be a valid integer."

Handling:
- Strip the invalid param from URL state
- Reset to defaults for that param
- Show toast: "Invalid filter removed. Showing default results."
- Do NOT show the full 422 error to the user
```

### Bulk Operation Error

```
╔══════════════════════════════════════════════════╗
║  Toast (error)                                    ║
║                                                  ║
║  Failed to update products                       ║
║  Some products not found or unauthorized. (403)  ║
║                                                  ║
╚══════════════════════════════════════════════════╝

The bulk action bar remains visible. Selection is preserved.
User can retry or modify selection.
```

### Bulk Action Feedback Flow

```
╔════════════════════════════════════════════════════════════════════════╗
║  BULK ACTION BUTTON LIFECYCLE                                         ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  1. DEFAULT:                                                           ║
║  ╔═══════════════════════════════════════════════════════════════╗    ║
║  ║  3 selected   [Activate] [Deactivate] [Delete]  [Cancel]     ║    ║
║  ╚═══════════════════════════════════════════════════════════════╝    ║
║                                                                        ║
║  2. MUTATION IN PROGRESS (all buttons disabled, spinner on active):    ║
║  ╔═══════════════════════════════════════════════════════════════╗    ║
║  ║  3 selected   [Activate] [⏳ Deactivating...] [Delete]       ║    ║
║  ║               (disabled)  (spinner, disabled)   (disabled)    ║    ║
║  ╚═══════════════════════════════════════════════════════════════╝    ║
║                                                                        ║
║  3. SUCCESS:                                                           ║
║  ┌──────────────────────────────────────────────┐                    ║
║  │ ✓ 3 products deactivated                      │  ← toast (green)  ║
║  └──────────────────────────────────────────────┘                    ║
║  - Bulk action bar hides (selection cleared)                           ║
║  - Affected rows fade out from current tab                             ║
║  - Status tab badge counts animate to new values                       ║
║                                                                        ║
║  4. PARTIAL SUCCESS:                                                   ║
║  ┌──────────────────────────────────────────────┐                    ║
║  │ ⚠️ 2 updated, 1 skipped                       │  ← toast (orange) ║
║  │ [Show Details ▼]                              │                    ║
║  └──────────────────────────────────────────────┘                    ║
║                                                                        ║
║  On expand:                                                            ║
║  ┌──────────────────────────────────────────────┐                    ║
║  │ ⚠️ 2 updated, 1 skipped                       │                    ║
║  │                                                │                    ║
║  │ Skipped:                                       │                    ║
║  │ · USB-C Hub: Cannot transition from draft      │                    ║
║  └──────────────────────────────────────────────┘                    ║
║                                                                        ║
║  5. FULL FAILURE:                                                      ║
║  ┌──────────────────────────────────────────────┐                    ║
║  │ ✕ Failed to update products                   │  ← toast (red)    ║
║  │   Not authorized (403)                        │                    ║
║  └──────────────────────────────────────────────┘                    ║
║  - Bulk action bar stays visible                                       ║
║  - Selection preserved for retry                                       ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Product Table Row Detail

Each row in the product table displays a `SellerProductMinimal` resource:

```
┌──┬─────────────────────────────────────────────────┬────────┬──────────┬──────┐
│[ ]│ Product                                         │ Status │ Price    │ SKUs │
├──┼─────────────────────────────────────────────────┼────────┼──────────┼──────┤
│  │                                                 │        │          │      │
│  │  ┌──────┐  Product Title Goes Here              │[Active]│ $12-$18  │ 6    │
│  │  │      │  Category > Subcategory > Leaf        │        │          │      │
│  │  │ IMG  │  Created: Feb 28, 2026                │        │          │ [...]│
│  │  │      │                                       │        │          │      │
│  │  └──────┘                                       │        │          │      │
│  │                                                 │        │          │      │
└──┴─────────────────────────────────────────────────┴────────┴──────────┴──────┘

Fields from SellerProductMinimal:
- image: Primary gallery image URL (thumbnail). NULL when product has no gallery -- show placeholder icon.
- title: Product title (truncated if long)
- shipping_category_id: Resolved to category name path via cached category tree
- status: Rendered as colored badge
- price_range: { min, max } -- display rules:
  - Both equal: "$24.99"
  - Different: "$12.00 - $18.00"
  - Both null (no SKUs, common for drafts): show "No price" in muted gray text
  - When region_id active: shows original_price range (not discount_price)
- sku_count: Number of SKUs. Show "0 SKUs" in muted text for products with no SKUs.
- created_at: Formatted date
- updated_at: Formatted date (shown in tooltip)

Null field display:

┌──┬──────────────────────────────────────────┬────────┬──────────┬──────┐
│  │                                          │        │          │      │
│  │  ┌──────┐  New Draft Product             │[Draft] │ No price │ 0    │
│  │  │  ┌─┐ │  Uncategorized                 │        │ (gray)   │      │
│  │  │  │?│ │  Created: Mar 1, 2026          │        │          │      │
│  │  │  └─┘ │                                │        │          │      │
│  │  └──────┘                                │        │          │      │
│  │   placeholder                            │        │          │      │
└──┴──────────────────────────────────────────┴────────┴──────────┴──────┘

The image placeholder is a gray box with a generic icon (package or image icon).
When price_range is null, the "No price" text uses text-muted-foreground color.
```

### Table Row Interaction States

```
╔════════════════════════════════════════════════════════════════════════╗
║  TABLE ROW STATES                                                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  DEFAULT (white bg):                                                   ║
║  ┌──┬──────────────────────────┬────────┬─────────┬──────┐           ║
║  │[ ]│ Product Title            │[Active]│ $24.99  │ 3    │           ║
║  └──┴──────────────────────────┴────────┴─────────┴──────┘           ║
║                                                                        ║
║  HOVER (gray-50 bg #f9fafb, context menu visible):                     ║
║  ┌──┬──────────────────────────┬────────┬─────────┬──────┐           ║
║  │[ ]│ Product Title            │[Active]│ $24.99  │ [...]│           ║
║  └──┴──────────────────────────┴────────┴─────────┴──────┘           ║
║  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░           ║
║  (light gray bg, three-dot [...] menu icon appears at right)           ║
║                                                                        ║
║  SELECTED (blue-50 bg, checkbox filled):                               ║
║  ┌──┬──────────────────────────┬────────┬─────────┬──────┐           ║
║  │[x]│ Product Title            │[Active]│ $24.99  │ 3    │           ║
║  └──┴──────────────────────────┴────────┴─────────┴──────┘           ║
║  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░           ║
║  (blue-50 bg #eff6ff, checkbox shows filled ✓)                         ║
║                                                                        ║
║  FOCUSED (keyboard navigation, 2px blue outline):                      ║
║  ┌──┬──────────────────────────┬────────┬─────────┬──────┐           ║
║  │[ ]│ Product Title            │[Active]│ $24.99  │ 3    ║           ║
║  └──┴──────────────────────────┴────────┴─────────┴──────┘           ║
║  ╔══════════════════════════════════════════════════════════╗         ║
║  (2px blue focus ring around entire row, visible for Tab nav)          ║
║                                                                        ║
║  Status tab states:                                                    ║
║  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐                ║
║  │  All     │ │  Active  │ │ Inactive │ │ Draft    │                ║
║  │  (47)    │ │   (23)   │ │   (8)    │ │   (5)    │                ║
║  └──────────┘ └──────────┘ └──────────┘ └──────────┘                ║
║   DEFAULT      ACTIVE       HOVER        DISABLED                      ║
║   (gray text,  (underline,  (gray bg,    (faded text,                  ║
║    no border)   bold text,   cursor       if count=0,                  ║
║                 blue         pointer)     cursor default)              ║
║                 underline)                                             ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Status Badge Colors

```
┌──────────┬───────────────┐
│ Status   │ Badge Style   │
├──────────┼───────────────┤
│ active   │ Green bg      │
│ inactive │ Gray bg       │
│ draft    │ Yellow bg     │
│ pending  │ Blue bg       │
│ violation│ Red bg        │
│ deleted  │ Dark gray bg  │
└──────────┴───────────────┘
```

### Status Badge Hover Tooltips

```
╔════════════════════════════════════════════════════════════════════════╗
║  STATUS BADGE TOOLTIP DETAILS                                         ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Each status badge shows contextual details on hover:                  ║
║                                                                        ║
║  [Active]                                                              ║
║     ┌──────────────────────────────────────┐                          ║
║     │ Live on storefront                    │  ← tooltip (dark bg)    ║
║     │ Updated 2h ago                        │                          ║
║     └──────────────────────────────────────┘                          ║
║                                                                        ║
║  [Violation]                                                           ║
║     ┌──────────────────────────────────────┐                          ║
║     │ Flagged: Missing required specs       │                          ║
║     │ Fix issues and resubmit for review    │                          ║
║     └──────────────────────────────────────┘                          ║
║                                                                        ║
║  [Pending]                                                             ║
║     ┌──────────────────────────────────────┐                          ║
║     │ Submitted Feb 28, 2026               │                          ║
║     │ Awaiting admin review                 │                          ║
║     └──────────────────────────────────────┘                          ║
║                                                                        ║
║  [Draft]                                                               ║
║     ┌──────────────────────────────────────┐                          ║
║     │ Not yet submitted                     │                          ║
║     │ Last saved 5 min ago                  │                          ║
║     └──────────────────────────────────────┘                          ║
║                                                                        ║
║  [Inactive]                                                            ║
║     ┌──────────────────────────────────────┐                          ║
║     │ Hidden from storefront                │                          ║
║     │ Deactivated Mar 1, 2026              │                          ║
║     └──────────────────────────────────────┘                          ║
║                                                                        ║
║  [Deleted]                                                             ║
║     ┌──────────────────────────────────────┐                          ║
║     │ Soft-deleted · Cannot be restored     │                          ║
║     │ Deleted Feb 27, 2026                 │                          ║
║     └──────────────────────────────────────┘                          ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## API Reference

### List Products

```
GET /api/seller/agent/seller-product/v1/seller-products

Query Parameters:
  response_type     nullable    "minimal" (frontend sends explicitly; backend default is "regular")
  status            nullable    draft | pending | active | inactive | violation | deleted
  shipping_category_id  nullable    UUID
  title             nullable    string (partial match)
  sort_by           nullable    title | created_at | updated_at | price (default: created_at)
  sort_order        nullable    asc | desc (default: desc)
  page              nullable    integer, min 1 (default: 1)
  per_page          nullable    integer, min 1, max 100 (default: 20)
  region_id         nullable    integer

Response (200):
{
  "object": "SellerProductCollection",
  "data": [
    {
      "object": "SellerProductMinimal",
      "id": 1,
      "title": "KingBank DDR5 RAM",
      "status": "active",
      "shipping_category_id": "9ce8d...",
      "image": "https://cdn.example.com/thumb.jpg",
      "price_range": { "min": 265.99, "max": 265.99 },
      "sku_count": 1,
      "created_at": "2026-02-28T16:48:10.000000Z",
      "updated_at": "2026-02-28T16:48:10.000000Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total": 47,
    "last_page": 3,
    "from": 1,
    "to": 20,
    "path": "...",
    "links": { "first": "...", "last": "...", "prev": null, "next": "..." }
  }
}
```

### Status Counts

```
GET /api/seller/agent/seller-product/v1/seller-products/status-counts

Response (200):
{
  "object": "SellerProductStatusCounts",
  "data": {
    "all": 47,
    "draft": 5,
    "pending": 4,
    "active": 23,
    "inactive": 8,
    "violation": 2,
    "deleted": 5
  }
}
```

### Bulk Status Update

```
POST /api/seller/agent/seller-product/v1/seller-products/bulk-status

Body:
{
  "product_ids": [1, 3, 4],
  "status": "inactive"
}

Valid target statuses: active, inactive, deleted
Max 50 product IDs per request.
Invalid transitions are skipped (not errors).

Response (200):
{
  "message": "Bulk status update completed.",
  "updated_count": 3,
  "skipped_count": 0,
  "skipped": []
}

Errors:
  403: Some products not found or unauthorized
  422: Empty/exceeds 50/invalid status/invalid ID
```

### Bulk Delete

```
POST /api/seller/agent/seller-product/v1/seller-products/bulk-delete

Body:
{
  "product_ids": [1, 3, 4]
}

Max 50 product IDs per request. Idempotent for already-deleted.

Response (200):
{
  "message": "Products deleted successfully.",
  "deleted_count": 3
}

Errors:
  403: Some products not found or unauthorized
  422: Empty/exceeds 50/invalid ID
```

---

## TanStack Query Keys

```
Query keys for product list feature:

["seller-products", "list", { status, title, shipping_category_id, sort_by, sort_order, page, per_page, region_id }]
["seller-products", "status-counts"]
["shipping-categories"]                    ← category tree for filter dropdown
["regions"]                                ← region list for region filter dropdown

Invalidation rules:
- After bulk-status:    invalidate ["seller-products", "list"] + ["seller-products", "status-counts"]
- After bulk-delete:    invalidate ["seller-products", "list"] + ["seller-products", "status-counts"]
- After single delete:  invalidate ["seller-products", "list"] + ["seller-products", "status-counts"]
- After create:         invalidate ["seller-products", "list"] + ["seller-products", "status-counts"]
- After update:         invalidate ["seller-products", "list"] + ["seller-products", "detail", id]

Stale times:
- seller-products list:   0 (always refetch on param change)
- status-counts:          0 (refetch on mount, invalidated after mutations)
- shipping-categories:    30 minutes (categories change rarely)
- regions:                30 minutes (regions change rarely)
```

---

## Notes

```
source_product_id:
  The SellerProductMinimal resource does NOT include source_product_id.
  A "Sourced from catalog" badge in list rows is not possible without
  a backend change to add this field to the minimal resource.
  source_product_id is only available on the full SellerProduct resource
  (response_type=regular or the Show endpoint).

Bulk operations and ETag:
  Bulk status update and bulk delete do NOT use ETag concurrency checks.
  Only individual PATCH uses If-Match / ETag. If two users bulk-update
  the same products simultaneously, both succeed on their valid targets.
  Invalid transitions are collected into the skipped array, not rejected.
```

---

## Accessibility

```
╔════════════════════════════════════════════════════════════════════════╗
║  ACCESSIBILITY REQUIREMENTS — PRODUCT LIST PAGE                       ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Status tabs:                                                          ║
║  - role="tablist" on the tab container                                ║
║  - role="tab" on each status tab                                      ║
║  - aria-selected="true" on the active tab                             ║
║  - aria-controls="{panel-id}" linking tab to panel                    ║
║                                                                        ║
║  Product table:                                                        ║
║  - <table> with <thead> and <tbody>                                   ║
║  - scope="col" on all header cells                                    ║
║  - Product rows are <tr> elements (not divs)                          ║
║                                                                        ║
║  Checkbox labels:                                                      ║
║  - Header checkbox: aria-label="Select all products on this page"     ║
║  - Row checkbox: aria-label="Select {product title}"                  ║
║                                                                        ║
║  Pagination:                                                           ║
║  - <nav> landmark with aria-label="Pagination"                        ║
║  - aria-current="page" on the current page button                     ║
║  - aria-label="Go to page {n}" on page links                         ║
║                                                                        ║
║  Bulk action bar:                                                      ║
║  - role="toolbar" with aria-label="Bulk actions for {n} products"     ║
║  - Each button has descriptive aria-label                              ║
║                                                                        ║
║  Filter result count:                                                  ║
║  - aria-live="polite" region announces "Showing {n} products"         ║
║    when filters change                                                 ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```
