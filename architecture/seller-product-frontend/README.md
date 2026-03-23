 # Seller Product Frontend Architecture

Frontend architecture documentation for the **Seller Product** feature -- product listing, creation, and editing in the MoveOn Global Seller Center.

---

## Documents

| # | Document | Description |
|---|----------|-------------|
| 1 | [Product List Page](./01-product-list-page.md) | Status tabs, search, filters, bulk operations, pagination, empty/loading/error states, accessibility |
| 2 | [Product Create Flow](./02-product-create-flow.md) | Multi-step creation: category picker (with recents), basic info, properties, SKUs, specifications, gallery, auto-save, preview |
| 3 | [Product Edit Flow](./03-product-edit-flow.md) | JSON Patch updates, ETag concurrency, conflict resolution (re-fetch flow), violation resubmission, full error catalog |
| 4 | [Variation & SKU Builder](./04-variation-sku-builder.md) | Property selection, value entry, SKU explosion guard, matrix generation, search/filter, per-SKU data, cascade behavior |
| 5 | [Gallery Manager](./05-gallery-manager.md) | Image/video management, asset picker integration, drag-and-drop reorder, primary image, video cards, accessibility |
| 6 | [Regional Pricing](./06-regional-pricing.md) | Per-region pricing, stock, availability per SKU, performance strategy, currency formatting, accessibility |
| 7 | [Data Flow & State Management](./07-data-flow-state-management.md) | TanStack Query caching, URL state, API types, price type conversion, error handling, form state |
| 8 | [Wholesale & Dropship Pricing](./08-wholesale-dropship-pricing.md) | 1688-model wholesale tiers (product-level), dropship pricing (product + SKU-level), schema, API, UI layout |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 16 (App Router) |
| Language | TypeScript 5 |
| API Client | Axios (service-based routing via `api.product.*`) |
| Server State | TanStack Query v5 |
| Forms | React Hook Form v7 + Zod validation |
| UI Components | shadcn/ui + Radix UI |
| Styling | Tailwind CSS v4 |
| Tables | TanStack Table v8 |
| Drag & Drop | @dnd-kit |
| i18n | next-intl |
| Toasts | Sonner |

---

## API Base URL

```
/api/seller/agent/seller-product/v1/seller-products
```

All requests scoped to authenticated seller's `agent_company_id` extracted from auth token.

---

## Backend API Reference

Full backend specification lives in:
`/daraz-seller-center-documentation/architecture/seller-product/`

### Endpoints Summary

| Method | Path | Description |
|--------|------|-------------|
| GET | `/` | List products (paginated, filtered, sorted). `response_type=minimal` for list view. |
| GET | `/{id}` | Show single product. `expand[]=` for relations. Returns `ETag` header. |
| POST | `/` | Create product. Returns 201 with `ETag` + `lock_version: 1`. |
| PATCH | `/{id}` | Update via RFC 6902 JSON Patch. Requires `Content-Type: application/json-patch+json` and `If-Match` ETag. |
| DELETE | `/{id}` | Soft delete (idempotent). Sets `status=deleted` + `deleted_at`. |
| POST | `/bulk-status` | Bulk status change (`active`, `inactive`, `deleted` only). Max 50. |
| POST | `/bulk-delete` | Bulk soft delete. Max 50. Idempotent. |
| GET | `/status-counts` | Counts per status (including deleted via `withTrashed`). |

---

## Related Architecture Documents

- [Media Center Frontend Architecture](../media-center-frontend-architecture.md) -- asset picker modal
- [Product Create/Edit Workflow](../product-create-edit-workflow.md) -- asset picker integration stub

---

## Quick Reference -- Status Lifecycle

```
  draft ──────► pending ──────► active ◄──────► inactive
                  │                                │
                  ▼                                │
              violation ──► pending (resubmit)     │
                                                    │
              any status ─────────────────────► deleted (soft)

  Seller can:    draft→pending, active→inactive, inactive→active,
                 violation→pending, any→deleted
  Admin only:    pending→active, pending→violation
  Bulk targets:  active, inactive, deleted only (max 50 per request)
```

---

## Quick Reference -- Key Constraints

```
┌──────────────────────────────────┬──────────────────────────────────┐
│ Resource                         │ Limit                            │
├──────────────────────────────────┼──────────────────────────────────┤
│ Properties per product           │ max 3                            │
│ Values per property              │ max 50                           │
│ SKUs per product                 │ max 100                          │
│ Specifications per product       │ max 50                           │
│ Gallery items (images + videos)  │ max 20                           │
│ Primary gallery image            │ exactly 1 (image type only)      │
│ Regional prices per SKU          │ 1 per region (unique sku+region) │
│ Wholesale tiers per product      │ 2 – 6 (product-level)            │
│ Dropship config per product      │ exactly 1 (product-level)        │
│ Dropship override per SKU        │ 0 or 1 (optional per SKU)        │
│ Bulk operation batch size        │ max 50 product IDs               │
│ List page per_page               │ max 100                          │
│ SKU price range                  │ 0.01 – 9,999,999,999.99         │
│ Title length                     │ 1 – 255 characters               │
│ Property/spec name length        │ max 100 characters               │
│ SKU code                         │ max 50, unique per product       │
│ Gallery URL                      │ max 500 characters               │
│ Description                      │ max 65,535 characters            │
└──────────────────────────────────┴──────────────────────────────────┘
```

---

## Quick Reference -- UI State Patterns

```
┌───────────────────────────────────────────────────────────────────────┐
│  FORM FIELD STATES                                                     │
│  Default -> Focus (blue ring) -> Valid (green check) -> Error (red)    │
│  -> Disabled (gray bg, not-allowed cursor)                             │
│                                                                        │
│  BUTTON STATES                                                         │
│  Default -> Hover (darker bg) -> Loading (spinner, disabled)           │
│  -> Success (check flash, 500ms) -> Disabled (gray, tooltip reason)   │
│                                                                        │
│  SELECTION PATTERNS                                                    │
│  Click: toggle single checkbox                                         │
│  Header checkbox: select/deselect all on current page                  │
│  Selected rows: blue-50 bg, bulk action bar appears                    │
└───────────────────────────────────────────────────────────────────────┘
```
