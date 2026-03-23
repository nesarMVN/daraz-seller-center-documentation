# Data Flow & State Management

## Overview

This document covers TanStack Query caching strategy, URL state management, API integration patterns, error handling, and form state management for the seller product feature.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│  BROWSER                                                            │
│                                                                     │
│  ┌───────────────────┐    ┌────────────────────┐                    │
│  │  URL State         │    │  React Hook Form   │                    │
│  │  (searchParams)    │    │  (form state)      │                    │
│  │                    │    │                    │                    │
│  │  ?status=active    │    │  title, desc,      │                    │
│  │  &page=2           │    │  properties,       │                    │
│  │  &sort_by=price    │    │  skus, gallery,    │                    │
│  │                    │    │  specs, regional   │                    │
│  └────────┬──────────┘    └────────┬───────────┘                    │
│           │                        │                                │
│           ▼                        ▼                                │
│  ┌────────────────────────────────────────────────┐                 │
│  │  TanStack Query (Server State Cache)           │                 │
│  │                                                │                 │
│  │  Query Keys:                                   │                 │
│  │  ["seller-products", "list", {filters}]        │                 │
│  │  ["seller-products", "detail", id, {expand}]   │                 │
│  │  ["seller-products", "status-counts"]          │                 │
│  │  ["shipping-categories"]                       │                 │
│  │  ["category-schemas"]                          │                 │
│  │  ["regions"]                                   │                 │
│  │                                                │                 │
│  │  Config:                                       │                 │
│  │  staleTime: 5 min (default)                    │                 │
│  │  gcTime: 10 min (default)                      │                 │
│  │  refetchOnWindowFocus: true                    │                 │
│  │  retry: 3                                      │                 │
│  └────────────────────┬───────────────────────────┘                 │
│                       │                                             │
│                       ▼                                             │
│  ┌────────────────────────────────────┐                             │
│  │  Axios API Client                  │                             │
│  │  api.product.*                     │                             │
│  │                                    │                             │
│  │  Base: NEXT_PUBLIC_PRODUCT_API_URL │                             │
│  │  Timeout: 30s                      │                             │
│  │  withCredentials: true             │                             │
│  └────────────────────┬───────────────┘                             │
│                       │                                             │
└───────────────────────┼─────────────────────────────────────────────┘
                        │
                        ▼
┌────────────────────────────────────────────────────────────┐
│  BACKEND API                                               │
│  /api/seller/agent/seller-product/v1/seller-products       │
│                                                            │
│  Auth: agent-api token (agent_company_id from JWT)         │
└────────────────────────────────────────────────────────────┘
```

---

## TanStack Query Configuration

### Query Keys

```
Query Key Structure                              Usage
──────────────────────────────────────────────────────────────────────

["seller-products", "list", {                    Product list page
  status, title, shipping_category_id,
  sort_by, sort_order, page, per_page,
  region_id                                      (optional, for regional filter)
}]

["seller-products", "detail", id, {              Product edit page
  expand: [...]                                  (no region_id — loads ALL regions)
}]

["seller-products", "status-counts"]             Status tab badges
                                                 Note: "all" count includes deleted

["shipping-categories"]                          Category picker tree

["category-schemas"]                             Variation + spec schemas
                                                 Keyed by category path string;
                                                 lookup by UUID via moveon.id

["regions"]                                      Regional pricing dropdowns
                                                 From Core API regions table
```

### Stale Time Configuration

```
┌─────────────────────────────────┬────────────┬────────────────────────┐
│ Query Key                       │ staleTime  │ Reason                 │
├─────────────────────────────────┼────────────┼────────────────────────┤
│ seller-products list            │ 0          │ Always refetch on nav  │
│ seller-products detail          │ 5 min      │ Edit page caches       │
│ seller-products status-counts   │ 30 sec     │ Frequent updates       │
│ shipping-categories             │ 30 min     │ Rarely changes         │
│ category-schemas                │ 30 min     │ Rarely changes         │
│ regions                         │ 60 min     │ Static reference data  │
└─────────────────────────────────┴────────────┴────────────────────────┘
```

### Cache Invalidation Rules

```
╔════════════════════════════════════════════════════════════════════════╗
║  INVALIDATION RULES                                                    ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  After CREATE (POST /seller-products):                                 ║
║  ├─ invalidate ["seller-products", "list"]  (all list queries)         ║
║  └─ invalidate ["seller-products", "status-counts"]                    ║
║                                                                        ║
║  After UPDATE (PATCH /seller-products/{id}):                           ║
║  ├─ invalidate ["seller-products", "list"]                             ║
║  ├─ invalidate ["seller-products", "status-counts"]                    ║
║  └─ setQueryData ["seller-products", "detail", id] (optimistic)        ║
║                                                                        ║
║  After DELETE (DELETE /seller-products/{id}):                           ║
║  ├─ invalidate ["seller-products", "list"]                             ║
║  ├─ invalidate ["seller-products", "status-counts"]                    ║
║  └─ removeQueries ["seller-products", "detail", id]  (evict cache)    ║
║                                                                        ║
║  After BULK STATUS (POST /seller-products/bulk-status):                ║
║  ├─ invalidate ["seller-products", "list"]                             ║
║  └─ invalidate ["seller-products", "status-counts"]                    ║
║                                                                        ║
║  After BULK DELETE (POST /seller-products/bulk-delete):                ║
║  ├─ invalidate ["seller-products", "list"]                             ║
║  └─ invalidate ["seller-products", "status-counts"]                    ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## URL State Management

### Product List Page

The product list page syncs all filter/sort/pagination state to URL search params. This enables shareable/bookmarkable URLs and browser back/forward navigation.

```
╔════════════════════════════════════════════════════════════════════════╗
║  URL <-> COMPONENT STATE SYNC                                          ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  URL params are the source of truth for list page state.               ║
║                                                                        ║
║  Read from URL:                                                        ║
║  const searchParams = useSearchParams()                                ║
║  const status = searchParams.get('status')                             ║
║  const title = searchParams.get('title')                               ║
║  const page = parseInt(searchParams.get('page') || '1')                ║
║  ...                                                                   ║
║                                                                        ║
║  Write to URL:                                                         ║
║  const router = useRouter()                                            ║
║  const updateParams = (updates) => {                                   ║
║    const params = new URLSearchParams(searchParams)                    ║
║    Object.entries(updates).forEach(([key, value]) => {                 ║
║      if (value === null) params.delete(key)                            ║
║      else params.set(key, value)                                       ║
║    })                                                                  ║
║    router.push(`/products?${params.toString()}`)                       ║
║  }                                                                     ║
║                                                                        ║
║  Flow:                                                                 ║
║                                                                        ║
║  User action (click tab, type search, change sort)                     ║
║       │                                                                ║
║       ▼                                                                ║
║  updateParams({ status: 'active', page: null })                        ║
║       │                                                                ║
║       ▼                                                                ║
║  URL changes: /products?status=active                                  ║
║       │                                                                ║
║       ▼                                                                ║
║  Component re-renders with new searchParams                            ║
║       │                                                                ║
║       ▼                                                                ║
║  TanStack Query detects new query key -> fetches                       ║
║       │                                                                ║
║       ▼                                                                ║
║  Data loads, table updates                                             ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Param Reset Rules

```
┌─────────────────────┬────────────────────────────────────────────┐
│ Action              │ Param Changes                              │
├─────────────────────┼────────────────────────────────────────────┤
│ Click status tab    │ Set status, reset page to 1                │
│ Type in search      │ Set title (debounced 300ms), reset page    │
│ Clear search        │ Remove title, reset page                   │
│ Select category     │ Set shipping_category_id, reset page       │
│ Clear category      │ Remove shipping_category_id, reset page    │
│ Change sort         │ Set sort_by + sort_order, reset page       │
│ Change page         │ Set page                                   │
│ Change per_page     │ Set per_page, reset page                   │
│ Select region       │ Set region_id, reset page                  │
│ Clear region        │ Remove region_id, reset page               │
└─────────────────────┴────────────────────────────────────────────┘

All filter changes reset page to 1 to avoid showing an empty page.
```

---

## Form State Management (Create/Edit)

### React Hook Form + Zod

```
╔════════════════════════════════════════════════════════════════════════╗
║  FORM STATE ARCHITECTURE                                               ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌──────────────────────────────┐                                     ║
║  │  React Hook Form             │                                     ║
║  │  useForm({                   │                                     ║
║  │    resolver: zodResolver(    │                                     ║
║  │      ProductFormSchema       │                                     ║
║  │    ),                        │                                     ║
║  │    defaultValues: {...}      │                                     ║
║  │  })                          │                                     ║
║  │                              │                                     ║
║  │  Tracks:                     │                                     ║
║  │  - All field values          │                                     ║
║  │  - Dirty state (isDirty)     │                                     ║
║  │  - Validation errors         │                                     ║
║  │  - Submit state              │                                     ║
║  └──────────────────┬───────────┘                                     ║
║                     │                                                  ║
║                     ▼                                                  ║
║  ┌──────────────────────────────┐                                     ║
║  │  Zod Schema Validation       │                                     ║
║  │                              │                                     ║
║  │  Validates:                  │                                     ║
║  │  - Field types               │                                     ║
║  │  - Required fields           │                                     ║
║  │  - Length constraints         │                                     ║
║  │  - Numeric ranges             │                                     ║
║  │  - Array sizes                │                                     ║
║  │  - Nested object shapes       │                                     ║
║  │  - Cross-field rules via     │                                     ║
║  │    .superRefine() (e.g.,     │                                     ║
║  │    discount <= original,      │                                     ║
║  │    SKU combo uniqueness,      │                                     ║
║  │    primary image count)       │                                     ║
║  │                              │                                     ║
║  │  Does NOT validate:           │                                     ║
║  │  - Category schema conformance│                                     ║
║  │    (required attrs, value     │                                     ║
║  │    options — server-only)     │                                     ║
║  └──────────────────────────────┘                                     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Form Field Arrays

Properties, SKUs, specifications, and gallery are managed as field arrays:

```
const { fields: propertyFields, append, remove, move } =
  useFieldArray({ control: form.control, name: 'properties' })

const { fields: skuFields, append: appendSku, remove: removeSku } =
  useFieldArray({ control: form.control, name: 'skus' })

const { fields: specFields, append: appendSpec, remove: removeSpec, move: moveSpec } =
  useFieldArray({ control: form.control, name: 'specifications' })

const { fields: galleryFields, append: appendGallery, remove: removeGallery, move: moveGallery } =
  useFieldArray({ control: form.control, name: 'gallery' })

The move() function handles drag-and-drop reorder:
  move(fromIndex, toIndex)  -> updates position values
```

### Edit Mode: Baseline + Diff

```
╔════════════════════════════════════════════════════════════════════════╗
║  CHANGE TRACKING FOR JSON PATCH GENERATION                             ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌──────────────────┐     ┌──────────────────┐                        ║
║  │  Baseline         │     │  Current Form    │                        ║
║  │  (from GET)       │     │  (user edits)    │                        ║
║  │                   │     │                  │                        ║
║  │  title: "Old"     │     │  title: "New"    │                        ║
║  │  properties: [A,B]│     │  properties: [B] │  (A removed)           ║
║  │  skus: [1,2,3]    │     │  skus: [1,2,3,4] │  (4 added)            ║
║  │  gallery: [x,y,z] │     │  gallery: [z,x,y] │  (reordered)         ║
║  └──────────────────┘     └──────────────────┘                        ║
║           │                        │                                   ║
║           └────────────┬───────────┘                                   ║
║                        │                                               ║
║                        ▼                                               ║
║           ┌──────────────────────┐                                     ║
║           │  Diff Engine          │                                     ║
║           │  generatePatchOps()   │                                     ║
║           └──────────┬───────────┘                                     ║
║                      │                                                 ║
║                      ▼                                                 ║
║           [                                                            ║
║             { op: "replace", path: "/title", value: "New" },           ║
║             { op: "remove", path: "/properties/101" },                 ║
║             { op: "add", path: "/skus/-", value: {...} },              ║
║             { op: "replace", path: "/gallery/403/position", value: 0 },║
║             { op: "replace", path: "/gallery/401/position", value: 1 },║
║             { op: "replace", path: "/gallery/402/position", value: 2 } ║
║           ]                                                            ║
║           (paths use BIGINT database IDs, not array indices)            ║
║                                                                        ║
║  The diff engine compares baseline vs current form state:              ║
║  - Top-level fields: direct comparison -> replace ops                  ║
║  - Array items: compare by ID                                          ║
║    - Items in current but not baseline -> add ops                      ║
║    - Items in baseline but not current -> remove ops                   ║
║    - Items in both: deep compare fields -> replace ops                 ║
║  - Position changes detected by comparing position values              ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Error Handling Strategy

### Error Categories

```
╔════════════════════════════════════════════════════════════════════════╗
║  ERROR HANDLING FLOW                                                   ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  API Error                                                             ║
║       │                                                                ║
║       ├─── 401 Unauthorized                                            ║
║       │    └─> Axios interceptor redirects to /login                   ║
║       │        (global handler, no per-feature code needed)            ║
║       │                                                                ║
║       ├─── 403 Forbidden                                               ║
║       │    └─> Toast: "You don't have permission for this action"      ║
║       │        (global interceptor shows toast)                        ║
║       │                                                                ║
║       ├─── 400 Bad Request                                              ║
║       │    └─> DB transaction failure: "Failed to create/update product.║
║       │        Please try again." Toast error, user retries.           ║
║       │                                                                ║
║       ├─── 404 Not Found                                               ║
║       │    ├─> Product list: shouldn't happen                          ║
║       │    ├─> Product detail: redirect to /products with toast        ║
║       │    │   "Product not found."                                    ║
║       │    ├─> Create (category): "Shipping category not found."       ║
║       │    └─> PATCH: product deleted while editing -> redirect        ║
║       │                                                                ║
║       ├─── 409 Conflict (test op failed)                               ║
║       │    └─> Toast with specific failure message                     ║
║       │                                                                ║
║       ├─── 412 Precondition Failed (ETag mismatch)                     ║
║       │    └─> Show conflict resolution dialog                         ║
║       │        (described in 03-product-edit-flow.md)                  ║
║       │                                                                ║
║       ├─── 415 Unsupported Media Type                                  ║
║       │    └─> Programming error. Toast: "Request format error"        ║
║       │                                                                ║
║       ├─── 422 Validation Error                                        ║
║       │    └─> Map errors to form fields (see below)                   ║
║       │                                                                ║
║       ├─── 428 Precondition Required                                   ║
║       │    └─> Programming error. Toast: "Missing If-Match header"     ║
║       │                                                                ║
║       └─── 500 Server Error                                            ║
║            └─> Toast: "Something went wrong. Please try again."        ║
║                (global interceptor)                                    ║
║                                                                        ║
║  Non-HTTP errors:                                                      ║
║       │                                                                ║
║       ├─── Network Timeout (Axios ECONNABORTED / timeout)              ║
║       │    └─> Toast: "Request timed out. Please try again."           ║
║       │        Axios timeout is 30s. On timeout, TanStack Query        ║
║       │        retries up to 3 times with exponential backoff.         ║
║       │                                                                ║
║       ├─── Offline / Connectivity (no response received)               ║
║       │    └─> Toast: "No internet connection. Check your network."    ║
║       │        Detect via navigator.onLine or Axios network error.     ║
║       │        TanStack Query pauses retries while offline             ║
║       │        (online manager) and resumes on reconnect.              ║
║       │                                                                ║
║       └─── 429 Too Many Requests (rate limiting)                       ║
║            └─> Toast: "Too many requests. Please wait a moment."       ║
║                Respect Retry-After header if present. TanStack Query   ║
║                retry handles this automatically with backoff.          ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Error State Visuals

```
╔════════════════════════════════════════════════════════════════════════╗
║  ERROR STATE VISUAL PATTERNS                                          ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  1. NETWORK ERROR (full-page, centered):                               ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │                                                                    │║
║  │                         [!]                                        │║
║  │                                                                    │║
║  │               Failed to load products                              │║
║  │                                                                    │║
║  │          Check your connection and try again                       │║
║  │                                                                    │║
║  │                      [Retry]                                       │║
║  │                                                                    │║
║  └──────────────────────────────────────────────────────────────────┘║
║  (centered icon + message + button, replaces table content)            ║
║                                                                        ║
║  2. FORM VALIDATION (inline field errors):                             ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  Product Title *                                                   │║
║  │  ┌─────────────────────────────────────────────────────────┐     │║
║  │  │                                                     [!] │     │║
║  │  └─────────────────────────────────────────────────────────┘     │║
║  │  The title field is required.                                      │║
║  │                                                                    │║
║  │  -> Page auto-scrolls to first error field                         │║
║  │  -> Error summary bar at top: "3 errors -- [Scroll to first]"      │║
║  └──────────────────────────────────────────────────────────────────┘║
║                                                                        ║
║  3. PERMISSION ERROR (toast notification):                             ║
║  ┌──────────────────────────────────────────────┐                    ║
║  │ X You don't have permission for this action   │  <- red bg toast   ║
║  │                                    (auto 5s)  │    auto-dismiss    ║
║  └──────────────────────────────────────────────┘                    ║
║                                                                        ║
║  4. SERVER ERROR (persistent toast):                                   ║
║  ┌──────────────────────────────────────────────┐                    ║
║  │ X Something went wrong                        │  <- red bg toast   ║
║  │   Please try again.          [Dismiss] [Retry]│    stays until     ║
║  └──────────────────────────────────────────────┘    dismissed        ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### 422 Validation Error Mapping

```
╔════════════════════════════════════════════════════════════════════════╗
║  MAPPING SERVER ERRORS TO FORM FIELDS                                  ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Server error shape:                                                   ║
║  {                                                                     ║
║    "message": "The given data was invalid.",                           ║
║    "errors": {                                                         ║
║      "title": ["The title field is required."],                        ║
║      "properties.0.name": ["Not defined in category schema."],         ║
║      "skus.2.price": ["Must be at least 0.01."],                       ║
║      "gallery": ["Exactly one image must be primary."],                ║
║      "specifications.0.value": ["Requires one of: X, Y, Z."]          ║
║    }                                                                   ║
║  }                                                                     ║
║                                                                        ║
║  Mapping function:                                                     ║
║                                                                        ║
║  function mapServerErrors(errors, setError) {                          ║
║    Object.entries(errors).forEach(([path, messages]) => {              ║
║      // Convert dot notation to React Hook Form path                   ║
║      // "properties.0.name" -> "properties.0.name"                     ║
║      // "skus.2.price" -> "skus.2.price"                               ║
║      // "gallery" -> "gallery" (root-level array error)                ║
║      setError(path, {                                                  ║
║        type: 'server',                                                 ║
║        message: messages[0]  // show first error message               ║
║      })                                                                ║
║    })                                                                  ║
║  }                                                                     ║
║                                                                        ║
║  For PATCH errors:                                                     ║
║  Server may return errors without dot-indexed paths:                   ║
║  - "Duplicate property name: 'Color'."  -> root properties error       ║
║  - "SKU '{id}' has incomplete property associations..." -> root skus    ║
║  - "Exactly one gallery image must be primary." -> root gallery         ║
║  Non-indexed errors map to the section root (shown as section banner). ║
║                                                                        ║
║  For list page (bulk ops):                                             ║
║  Errors don't map to form fields. Show toast with message.             ║
║  Bulk status returns skipped items in response body:                   ║
║  { updated_count: 8, skipped_count: 2, skipped: [{ id, reason }] }    ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## API Client Integration

### Axios Service Routing

```
All product API calls go through the product service client:

api.product.get('/seller-products', { params })
  // List: params includes response_type: 'minimal'

api.product.get(`/seller-products/${id}`, { params })
  // Detail: params includes expand[], response_type: 'regular'

api.product.post('/seller-products', body)
  // Create: standard JSON Content-Type

api.product.patch(`/seller-products/${id}`, body, {
  headers: {
    'Content-Type': 'application/json-patch+json',  // REQUIRED for PATCH
    'If-Match': `"${etag}"`                          // REQUIRED ETag
  }
})

api.product.delete(`/seller-products/${id}`)
  // Soft delete (idempotent). No body or If-Match needed.

api.product.post('/seller-products/bulk-status', body)
  // body: { product_ids: int[], status: 'active'|'inactive'|'deleted' }

api.product.post('/seller-products/bulk-delete', body)
  // body: { product_ids: int[] }

api.product.get('/seller-products/status-counts')
  // No params. Returns counts for all statuses including deleted.

The api.product prefix routes to NEXT_PUBLIC_PRODUCT_API_URL.
Auth token is attached by request interceptor (when implemented).
```

### Request/Response Types

```typescript
// Query params for list endpoint
interface SellerProductListParams {
  response_type?: 'minimal' | 'regular'
  status?: 'draft' | 'pending' | 'active' | 'inactive' | 'violation' | 'deleted'
  shipping_category_id?: string  // UUID
  title?: string                 // partial match (LIKE)
  sort_by?: 'title' | 'created_at' | 'updated_at' | 'price'
  sort_order?: 'asc' | 'desc'
  page?: number                  // min 1
  per_page?: number              // min 1, max 100, default 20
  region_id?: number             // filters to products with active SKUs in region
}

// List response wrapper
interface SellerProductListResponse {
  object: 'SellerProductCollection'
  data: SellerProductMinimal[]
  pagination: {
    current_page: number
    per_page: number
    total: number
    last_page: number
    from: number
    to: number
    path: string
    links: { first: string; last: string; prev: string | null; next: string | null }
  }
}

// Minimal product (list view)
interface SellerProductMinimal {
  object: 'SellerProductMinimal'
  id: number
  title: string
  status: string
  shipping_category_id: string
  image: string | null            // primary gallery image URL, null if no primary
  price_range: { min: number; max: number } | null  // null when no SKUs
  sku_count: number
  created_at: string              // ISO 8601 with microseconds
  updated_at: string
}

// Full product (detail/edit view)
interface SellerProduct {
  object: 'SellerProduct'
  id: number
  title: string
  description: string | null
  status: string
  shipping_category_id: string
  source_product_id: string | null  // ULID, read-only after creation
  created_by: number
  etag: string                      // MD5 hash, also in ETag response header
  lock_version: number              // monotonic counter
  created_at: string
  updated_at: string
  properties?: { data: SellerProductProperty[] }
  skus?: { data: SellerProductSku[] }
  specifications?: { data: SellerProductSpecification[] }
  gallery?: { data: SellerProductGalleryItem[] }
  category?: { id: string; name: string; path: string[] }
}

// Nested resources (populated via expand[])
interface SellerProductProperty {
  object: 'SellerProductProperty'
  id: number
  name: string                      // max 100
  position: number
  values: { data: SellerProductPropertyValue[] }
}

interface SellerProductPropertyValue {
  object: 'SellerProductPropertyValue'
  id: number
  value: string                     // max 100
  image_url: string | null          // url, max 500
  position: number
}

interface SellerProductSku {
  object: 'SellerProductSku'
  id: number
  sku_code: string | null           // max 50, unique per product
  price: string                     // DECIMAL(12,2) as string e.g. "24.99"
  stock: number
  is_active: boolean
  // NOTE: No position field. SKU display order = array index in response.
  // Reorder is persisted via array ordering in POST body or order of add ops in PATCH.
  associations: { data: SellerProductSkuAssociation[] }
  regional_prices?: { data: SellerProductSkuRegionPrice[] }  // when expand[]=regional_prices
}

interface SellerProductSkuAssociation {
  object: 'SellerProductSkuAssociation'
  id: number
  property_id: number
  property_value_id: number
  property: { id: number; name: string; position: number }
  property_value: { id: number; value: string; image_url: string | null; position: number }
}

interface SellerProductSkuRegionPrice {
  object: 'SellerProductSkuRegionPrice'
  id: number
  region_id: number                 // FK to Core API regions table
  original_price: string            // DECIMAL(12,2) as string
  discount_price: string | null     // null = no discount
  stock: number
  is_active: boolean
}

interface SellerProductSpecification {
  object: 'SellerProductSpecification'
  id: number
  name: string                      // max 100
  value: string                     // max 500
  type: 'text' | 'select'
  position: number
}

interface SellerProductGalleryItem {
  object: 'SellerProductGalleryItem'
  id: number
  asset_id: string | null           // UUID, FK to assets (ON DELETE SET NULL)
  url: string                       // url, max 500
  type: 'image' | 'video'
  thumbnail_url: string | null      // required when type=video
  duration: number | null           // seconds, for video
  position: number
  is_primary: boolean               // only for type=image
}

// Status counts
interface SellerProductStatusCounts {
  object: 'SellerProductStatusCounts'
  data: {
    all: number      // includes soft-deleted
    draft: number
    pending: number
    active: number
    inactive: number
    violation: number
    deleted: number
  }
}

// Type Conversion — Price String/Number Reconciliation
//
// The API GET response returns prices as `string` (DECIMAL serialization):
//   sku.price = "24.99"
//   regional_prices[].original_price = "2499.00"
//   regional_prices[].discount_price = "149.00" | null
//
// The form uses `number` (Zod validates with z.number()):
//   SkuSchema.price = z.number().min(0.01)
//   RegionalPriceSchema.original_price = z.number().min(0.01)
//
// Both are correct for their context. Conversion rules:
//
// On load (API -> form):
//   parseFloat(sku.price)                      // "24.99" -> 24.99
//   parseFloat(regionalPrice.original_price)   // "2499.00" -> 2499.00
//   regionalPrice.discount_price !== null
//     ? parseFloat(regionalPrice.discount_price) : null
//
// On display (form -> UI):
//   Intl.NumberFormat(locale, { style: 'currency', currency }).format(value)
//
// On submit (form -> API):
//   POST body sends numbers directly (24.99). Server accepts numeric types.
//   PATCH replace ops send numbers: { op: "replace", path: "/skus/{id}/price", value: 29.99 }
//
// See 02-product-create-flow.md Zod schema section for the form-side validation.

// JSON Patch operation
interface PatchOperation {
  op: 'add' | 'remove' | 'replace' | 'test'
  path: string
  value?: unknown  // required for add, replace, test; omitted for remove
}
```

---

## Loading State Patterns

```
╔════════════════════════════════════════════════════════════════════════╗
║  LOADING STATE PATTERNS                                                ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Product List (response_type: 'minimal'):                               ║
║  isLoading (first load)  -> Full page skeleton                         ║
║  isFetching (refetch)    -> Subtle loading indicator in table header   ║
║  isError                 -> Error state with retry button              ║
║                                                                        ║
║  Status Counts:                                                        ║
║  isLoading               -> Tab badges show "..." or skeleton          ║
║  isFetching              -> No visual indicator (background update)    ║
║                                                                        ║
║  Product Detail (response_type: 'regular', expand[]=all):              ║
║  isLoading               -> Full page skeleton matching form layout    ║
║  isFetching              -> No visual indicator                        ║
║  isError + 404           -> Redirect to /products                      ║
║  Note: expand[]=regional_prices implicitly includes skus.              ║
║                                                                        ║
║  Category Tree:                                                        ║
║  isLoading               -> Spinner in category picker dropdown        ║
║  isError                 -> "Failed to load categories" with retry     ║
║                                                                        ║
║  Mutations (create/update/delete/bulk):                                ║
║  isPending               -> Disable submit button, show spinner        ║
║  isError                 -> Re-enable button, show error               ║
║  isSuccess               -> Toast, navigate/invalidate                 ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Visual Loading State Examples

```
╔════════════════════════════════════════════════════════════════════════╗
║  LOADING STATE VISUAL PATTERNS                                        ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  INITIAL LOAD (no data, full skeleton):                                ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐            │║
║  │  │ --- (--) │ │ --- (--) │ │ --- (--) │ │ --- (--) │            │║
║  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘            │║
║  │                                                                    │║
║  │  ┌──┬──────────────────────────┬────────┬─────────┬──────┐       │║
║  │  │--│ ------------------------ │ ------ │ ------- │ ----│       │║
║  │  │--│ ------------------------ │ ------ │ ------- │ ----│       │║
║  │  │--│ ------------------------ │ ------ │ ------- │ ----│       │║
║  │  └──┴──────────────────────────┴────────┴─────────┴──────┘       │║
║  └──────────────────────────────────────────────────────────────────┘║
║  (no data rendered, all elements are skeleton placeholders)            ║
║                                                                        ║
║  BACKGROUND REFETCH (data visible, subtle indicator):                  ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  ═══════════════════════════════════════════════                   │║
║  │  <- thin progress bar at top of table (blue, animated)            │║
║  │                                                                    │║
║  │  ┌──┬─────────────────────┬────────┬─────────┐  (existing data   │║
║  │  │[ ]│ KingBank DDR5 RAM   │ Active │ $265.99 │   visible at     │║
║  │  │[ ]│ Cotton T-Shirt      │ Draft  │ $12-$18 │   60% opacity)   │║
║  │  │[ ]│ Wireless Mouse      │ Active │ $24.99  │                  │║
║  │  └──┴─────────────────────┴────────┴─────────┘                  │║
║  └──────────────────────────────────────────────────────────────────┘║
║                                                                        ║
║  MUTATION IN PROGRESS (partial UI disabled):                           ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  ┌──┬─────────────────────┬────────┬─────────┐                  │║
║  │  │[x]│ KingBank DDR5 RAM   │ Active │ $265.99 │  <- dimmed        │║
║  │  │[ ]│ Cotton T-Shirt      │ Draft  │ $12-$18 │  <- normal        │║
║  │  │[x]│ Wireless Mouse      │ Active │ $24.99  │  <- dimmed        │║
║  │  └──┴─────────────────────┴────────┴─────────┘                  │║
║  │                                                                    │║
║  │  ╔═══════════════════════════════════════════════════════════╗    │║
║  │  ║ 2 selected  [Deactivating...]                             ║    │║
║  │  ╚═══════════════════════════════════════════════════════════╝    │║
║  └──────────────────────────────────────────────────────────────────┘║
║  (selected rows dimmed, button shows spinner, rest of UI interactive)  ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Drag and Drop Integration

All drag-and-drop uses @dnd-kit. The pattern is consistent across gallery, property values, specifications, and SKU rows.

```
╔════════════════════════════════════════════════════════════════════════╗
║  @DND-KIT INTEGRATION PATTERN                                          ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Shared setup for all sortable lists:                                  ║
║                                                                        ║
║  <DndContext                                                           ║
║    sensors={[useSensor(PointerSensor, {                                ║
║      activationConstraint: { distance: 8 }                             ║
║    })]}                                                                ║
║    collisionDetection={closestCenter}                                  ║
║    onDragEnd={handleDragEnd}                                           ║
║  >                                                                     ║
║    <SortableContext                                                     ║
║      items={items.map(i => i.id)}                                      ║
║      strategy={verticalListSortingStrategy}                            ║
║    >                                                                   ║
║      {items.map(item => (                                              ║
║        <SortableItem key={item.id} id={item.id}>                       ║
║          {renderItem(item)}                                            ║
║        </SortableItem>                                                 ║
║      ))}                                                               ║
║    </SortableContext>                                                   ║
║    <DragOverlay>                                                       ║
║      {activeItem && renderItem(activeItem)}                            ║
║    </DragOverlay>                                                      ║
║  </DndContext>                                                         ║
║                                                                        ║
║  handleDragEnd:                                                        ║
║  1. Find old and new index from event                                  ║
║  2. Call form.move(oldIndex, newIndex) for field array reorder         ║
║  3. Recalculate position values (0, 1, 2, ...)                         ║
║                                                                        ║
║  Where drag-and-drop is used:                                          ║
║  ┌──────────────────────┬──────────────────────────────────────────┐  ║
║  │ Feature              │ What is reordered                        │  ║
║  ├──────────────────────┼──────────────────────────────────────────┤  ║
║  │ Gallery              │ Image/video cards (position field)       │  ║
║  │ Property values      │ Value tags within a property (position)  │  ║
║  │ Specifications       │ Spec rows (position field)               │  ║
║  │ SKU rows             │ SKU table rows (display order)           │  ║
║  └──────────────────────┴──────────────────────────────────────────┘  ║
║                                                                        ║
║  Visual feedback:                                                      ║
║  - Drag handle (= icon) on the left of each sortable item             ║
║  - DragOverlay shows a semi-transparent copy of the dragged item       ║
║  - Drop zones highlight on hover                                       ║
║  - Smooth animation via CSS transitions on position changes            ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Keyboard Navigation & Accessibility

```
╔════════════════════════════════════════════════════════════════════════╗
║  KEYBOARD PATTERNS                                                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Product List Page:                                                    ║
║  ┌─────────────────────┬────────────────────────────────────────────┐ ║
║  │ Key                 │ Action                                     │ ║
║  ├─────────────────────┼────────────────────────────────────────────┤ ║
║  │ Tab                 │ Move focus through toolbar, table, pagination║
║  │ Enter               │ Open focused product (navigate to edit)    │ ║
║  │ Space               │ Toggle checkbox on focused row             │ ║
║  │ Escape              │ Close any open dropdown/modal              │ ║
║  │ Arrow Up/Down       │ Navigate rows in the product table         │ ║
║  └─────────────────────┴────────────────────────────────────────────┘ ║
║                                                                        ║
║  Create/Edit Form:                                                    ║
║  ┌─────────────────────┬────────────────────────────────────────────┐ ║
║  │ Key                 │ Action                                     │ ║
║  ├─────────────────────┼────────────────────────────────────────────┤ ║
║  │ Tab                 │ Move between form fields                   │ ║
║  │ Shift+Tab           │ Move backwards between fields              │ ║
║  │ Enter               │ Add property value (in tag input)          │ ║
║  │ Escape              │ Close modal (category picker, asset picker)│ ║
║  │ Ctrl+S / Cmd+S      │ Save form (draft or changes)               │ ║
║  └─────────────────────┴────────────────────────────────────────────┘ ║
║                                                                        ║
║  Drag and Drop (@dnd-kit keyboard sensor):                            ║
║  ┌─────────────────────┬────────────────────────────────────────────┐ ║
║  │ Key                 │ Action                                     │ ║
║  ├─────────────────────┼────────────────────────────────────────────┤ ║
║  │ Space (on handle)   │ Pick up / drop item                        │ ║
║  │ Arrow Up/Down       │ Move item up/down while dragging           │ ║
║  │ Arrow Left/Right    │ Move item left/right (gallery grid)        │ ║
║  │ Escape              │ Cancel drag operation                      │ ║
║  └─────────────────────┴────────────────────────────────────────────┘ ║
║                                                                        ║
║  @dnd-kit sensors configured:                                         ║
║  - PointerSensor: mouse/touch with activationConstraint distance: 8   ║
║  - KeyboardSensor: keyboard-only drag via Space + Arrow keys           ║
║                                                                        ║
║  ARIA attributes:                                                     ║
║  - Sortable items: role="listitem", aria-roledescription="sortable"   ║
║  - Drag handle: aria-label="Reorder {item name}"                      ║
║  - Live region announces position changes during drag                 ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Touch Support

```
╔════════════════════════════════════════════════════════════════════════╗
║  TOUCH / MOBILE CONSIDERATIONS                                        ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Drag and drop on touch devices:                                      ║
║  - @dnd-kit TouchSensor with activationConstraint delay: 250ms        ║
║  - Prevents accidental drags while scrolling                          ║
║  - Visual indicator when long-press activates drag                    ║
║                                                                        ║
║  SKU matrix on small screens:                                         ║
║  - Horizontal scroll on the table container                           ║
║  - Sticky first column (property values) for reference                ║
║  - Column priority: Property values > Price > Stock > Active          ║
║                                                                        ║
║  Gallery grid on small screens:                                       ║
║  - 2 columns instead of 5                                             ║
║  - Drag still works via touch sensor                                  ║
║                                                                        ║
║  Regional pricing on small screens:                                   ║
║  - Stacked card layout instead of table rows                          ║
║  - Each region renders as a card with inline fields                   ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Route Structure

```
/[locale]/(seller)/
├── products/
│   ├── page.tsx                    <- Product list page
│   ├── create/
│   │   └── page.tsx                <- Product create form
│   └── [id]/
│       └── edit/
│           └── page.tsx            <- Product edit form
```

All routes use the seller layout with sidebar navigation.
The `(seller)` route group applies the seller layout with `SidebarProvider`, `AppSidebar`, and `SiteHeader`.

---

## i18n Keys

```
Namespace: "products"

products.page_title              "Products"
products.create_title            "Create Product"
products.edit_title              "Edit Product"
products.tabs.all                "All"
products.tabs.active             "Active"
products.tabs.inactive           "Inactive"
products.tabs.draft              "Draft"
products.tabs.pending            "Pending"
products.tabs.violation          "Violation"
products.tabs.deleted            "Deleted"
products.search_placeholder      "Search products..."
products.no_products             "You have no products yet"
products.no_results              "No products match your search"
products.create_button           "New Product"
products.save_draft              "Save as Draft"
products.submit_review           "Submit for Review"
products.save_changes            "Save Changes"
products.delete_confirm          "Delete {count} Products?"
products.deactivate_confirm      "Deactivate {count} Products?"
products.conflict_title          "Save Conflict"
products.unsaved_changes         "You have unsaved changes"
products.toast.created           "Product saved as draft"
products.toast.submitted         "Product submitted for review"
products.toast.updated           "Product saved"
products.toast.deleted           "{count} products deleted"
products.toast.bulk_status       "{updated} updated, {skipped} skipped"
products.toast.conflict          "Product was modified by another user"
products.toast.not_found         "Product not found"
products.toast.save_failed       "Save failed. Please try again."
products.toast.format_error      "Request format error. Please reload."
```
