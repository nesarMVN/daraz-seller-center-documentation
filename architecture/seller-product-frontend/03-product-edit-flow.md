# Product Edit Flow

## Overview

Product editing uses RFC 6902 JSON Patch with RFC 7232 ETag-based optimistic concurrency. The edit page loads the existing product, displays it in the same form layout as create, and tracks changes. On save, only the changed fields are sent as JSON Patch operations, guarded by the `If-Match` ETag header.

**Route:** `/[locale]/(seller)/products/{id}/edit`

---

## Page Layout

Same form layout as the create page, but with all fields pre-populated from the existing product data.

```
╔═════════════════════════════════════════════════════════════════════════════╗
║  MoveOn Seller Center                              John Doe   [Bell] [Cog]  ║
╠═════════════════════════════════════════════════════════════════════════════╣
║                                                                             ║
║  Products > Edit Product                                                    ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  STATUS BADGE + SOURCE INFO                                         │   ║
║  │                                                                     │   ║
║  │  Status: [Active]       Created: Feb 28, 2026                       │   ║
║  │  Source: 01KJJJD0M7... (sourced from vendor catalog)                │   ║
║  │  ETag: a1b2c3d4e5f6...   Lock Version: 3                          │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  SECTION 1: CATEGORY                                                │   ║
║  │  Electronics & Gadgets > Computer Peripherals > Mouse          [x]  │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  SECTION 2: BASIC INFORMATION                                       │   ║
║  │  Title: [Wireless Gaming Mouse 2.4GHz RGB Ergonomic___________]     │   ║
║  │  Description: [Rich text editor with existing content]              │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  SECTION 3: VARIATIONS & SKUs (populated from existing data)        │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  SECTION 4: SPECIFICATIONS (populated)                              │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  SECTION 5: GALLERY (populated)                                     │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  SECTION 6: REGIONAL PRICING (populated)                            │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  ACTION BAR (sticky bottom)                                         │   ║
║  │                                                                     │   ║
║  │  [Cancel]           [Save as Draft]  [Save Changes]  [Submit]       │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

Action buttons depend on product status:

```
┌────────────┬──────────────────────────────────────────────────┐
│ Status     │ Available Actions                                │
├────────────┼──────────────────────────────────────────────────┤
│ draft      │ [Cancel] [Save Draft] [Submit for Review]        │
│ pending    │ [Cancel] [Save Changes]                          │
│ active     │ [Cancel] [Save Changes] [Deactivate]             │
│ inactive   │ [Cancel] [Save Changes] [Activate]               │
│ violation  │ [Cancel] [Save Changes] [Resubmit]               │
│ deleted    │ (read-only -- no action buttons)                 │
└────────────┴──────────────────────────────────────────────────┘
```

### Violation Resubmission Workflow

```
╔════════════════════════════════════════════════════════════════════════╗
║  VIOLATION STATUS — FIX AND RESUBMIT                                  ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  When status === "violation", the edit page shows a violation banner:  ║
║                                                                        ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │  This product was flagged for a policy violation.                 │ ║
║  │                                                                   │ ║
║  │  Reason: Missing required specifications for this category.       │ ║
║  │  (or generic: "This product does not meet listing requirements.") │ ║
║  │                                                                   │ ║
║  │  Fix the issues below and click [Resubmit] to request review.    │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║  (red-50 bg, red left border, violation icon)                          ║
║                                                                        ║
║  The violation reason is sourced from the API response if available    ║
║  (future: violation_reason field). If not present, a generic message   ║
║  is shown.                                                             ║
║                                                                        ║
║  Resubmission flow:                                                    ║
║  1. Seller reviews and fixes the flagged issues.                       ║
║  2. Seller clicks [Resubmit] in the action bar.                        ║
║  3. Frontend validates all mandatory fields are filled                  ║
║     (same validation as Submit for Review in create flow).             ║
║  4. PATCH includes status transition: violation -> pending              ║
║     { "op": "replace", "path": "/status", "value": "pending" }        ║
║     Combined with any other edits in the same patch.                   ║
║  5. On success: toast "Product resubmitted for review",                ║
║     banner disappears, status badge updates to "Pending".              ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Data Loading

### Fetch Product for Edit

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: Page loads, fetch product with all relations                 ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  API Call:                                                             ║
║  GET /api/seller/agent/seller-product/v1/seller-products/{id}          ║
║      ?expand[]=properties                                              ║
║      &expand[]=skus                                                    ║
║      &expand[]=specifications                                          ║
║      &expand[]=gallery                                                 ║
║      &expand[]=category                                                ║
║      &expand[]=regional_prices                                         ║
║      &response_type=regular                                            ║
║                                                                        ║
║  Note: expand[]=regional_prices implicitly includes expand[]=skus.     ║
║  No region_id param is passed -- this loads ALL regions for all SKUs.  ║
║  (Passing region_id would filter to a single region per SKU.)          ║
║  Soft-deleted products can also be fetched via withTrashed().          ║
║                                                                        ║
║  Response includes ETag header:                                        ║
║  ETag: "a1b2c3d4e5f6789012345678abcdef01"                          ║
║                                                                        ║
║  Response body (200):                                                  ║
║  {                                                                     ║
║    "data": {                                                           ║
║      "object": "SellerProduct",                                        ║
║      "id": 42,                                                         ║
║      "title": "Wireless Gaming Mouse...",                              ║
║      "description": "<p>Professional...</p>",                          ║
║      "status": "active",                                               ║
║      "shipping_category_id": "9ce8d860-...",                           ║
║      "source_product_id": "01KJJJD0M7E89...",                          ║
║      "created_by": 7,                                                  ║
║      "etag": "a1b2c3d4e5f678...",                                      ║
║      "lock_version": 3,                                                ║
║      "created_at": "2026-02-28T10:00:00.000000Z",                     ║
║      "updated_at": "2026-02-28T14:30:00.000000Z",                     ║
║      "properties": { "data": [                                         ║
║        { "object": "SellerProductProperty", "id": 101,                ║
║          "name": "Color", "position": 0,                               ║
║          "values": { "data": [                                         ║
║            { "object": "SellerProductPropertyValue", "id": 201,       ║
║              "value": "Black", "image_url": null, "position": 0 }    ║
║          ]}                                                            ║
║        }                                                               ║
║      ]},                                                               ║
║      "skus": { "data": [                                              ║
║        { "object": "SellerProductSku", "id": 301,                     ║
║          "sku_code": "BLK-01", "price": "24.99", "stock": 100,       ║
║          "is_active": true,                                            ║
║          "associations": { "data": [                                   ║
║            { "id": 401, "property_id": 101,                           ║
║              "property_value_id": 201,                                 ║
║              "property": { "id": 101, "name": "Color" },             ║
║              "property_value": { "id": 201, "value": "Black" } }     ║
║          ]},                                                           ║
║          "regional_prices": { "data": [                                ║
║            { "object": "SellerProductSkuRegionPrice", "id": 501,      ║
║              "region_id": 3, "original_price": "149.00",              ║
║              "discount_price": null, "stock": 50, "is_active": true } ║
║          ]}                                                            ║
║        }                                                               ║
║      ]},                                                               ║
║      "specifications": { "data": [...] },                              ║
║      "gallery": { "data": [                                           ║
║        { "object": "SellerProductGalleryItem", "id": 601,             ║
║          "asset_id": "uuid-or-null", "url": "https://...",            ║
║          "type": "image", "thumbnail_url": null, "duration": null,    ║
║          "position": 0, "is_primary": true }                          ║
║      ]},                                                               ║
║      "category": { "id": "...", "name": "Mouse", "path": [...] }      ║
║    }                                                                   ║
║  }                                                                     ║
║                                                                        ║
║  Error (404):                                                          ║
║  { "message": "Product not found." }                                   ║
║  Redirect to /products with toast. Triggers when ID does not exist     ║
║  or belongs to another company.                                        ║
║                                                                        ║
║  Store etag in component state for later PATCH requests.               ║
║  Populate all form fields from response data.                          ║
║                                                                        ║
║  If status === "deleted":                                              ║
║    Show read-only banner: "This product has been deleted."             ║
║    All form fields disabled. No action buttons.                        ║
║    deleted_at timestamp shown in status badge area.                    ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Loading State

```
┌─────────────────────────────────────────────────────────────────────┐
│  Products > Edit Product                                            │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  ░░░░░░░░░░░░  ░░░░░░░░░░░░░░░░░░░░░░░░                    │  │
│  │                                                               │  │
│  │  ┌─────────────────────────────────────────────────────────┐ │  │
│  │  │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │ │  │
│  │  │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │ │  │
│  │  └─────────────────────────────────────────────────────────┘ │  │
│  │                                                               │  │
│  │  ┌─────────────────────────────────────────────────────────┐ │  │
│  │  │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │ │  │
│  │  │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │ │  │
│  │  │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │ │  │
│  │  └─────────────────────────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  Skeleton loaders match the form section layout.                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Change Tracking and JSON Patch Generation

The edit form tracks all changes the user makes against the original loaded data. On save, it generates a minimal JSON Patch document containing only the operations needed.

### Change Detection Strategy

```
╔════════════════════════════════════════════════════════════════════════╗
║  CHANGE TRACKING MODEL                                                ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  On page load:                                                         ║
║  - Store deep clone of original product data as "baseline"             ║
║  - Store ETag value                                                    ║
║                                                                        ║
║  As user edits:                                                        ║
║  - React Hook Form tracks form state changes                          ║
║  - Each edit is tracked but NOT sent to server yet                     ║
║                                                                        ║
║  On save:                                                              ║
║  - Compare current form state vs baseline                              ║
║  - Generate JSON Patch operations for each difference                  ║
║                                                                        ║
║  Diff categories and their generated ops:                              ║
║                                                                        ║
║  Top-level fields:                                                     ║
║    title "Old" -> "New"                                                ║
║    => { op: "replace", path: "/title", value: "New" }                  ║
║    (same pattern for /description, /shipping_category_id)              ║
║                                                                        ║
║  Properties:                                                           ║
║    New property added:                                                 ║
║    => { op: "add", path: "/properties/-",                              ║
║         value: { name: "Color", position: 0,                           ║
║                  values: [{ value: "Red", position: 0 }] } }           ║
║                                                                        ║
║    Property removed (cascades to values + SKU associations):           ║
║    => { op: "remove", path: "/properties/{id}" }                       ║
║                                                                        ║
║    Property renamed:                                                   ║
║    => { op: "replace", path: "/properties/{id}/name",                  ║
║         value: "Colour" }                                              ║
║                                                                        ║
║    Property value text changed:                                        ║
║    => { op: "replace", path: "/properties/{id}/values/{id}/value",     ║
║         value: "New Value" }                                           ║
║                                                                        ║
║    Property value image changed:                                       ║
║    => { op: "replace", path: "/properties/{id}/values/{id}/image_url", ║
║         value: "https://cdn.../new.jpg" }                              ║
║                                                                        ║
║  SKUs:                                                                 ║
║    New SKU added:                                                      ║
║    => { op: "add", path: "/skus/-", value: { price: 24.99,            ║
║         stock: 100, sku_code: "RD-800", is_active: true,              ║
║         property_value_ids: [205],                                     ║
║         property_value_refs: ["temp-001"] } }                          ║
║                                                                        ║
║    SKU removed:                                                        ║
║    => { op: "remove", path: "/skus/{id}" }                             ║
║                                                                        ║
║    SKU price changed:                                                  ║
║    => { op: "replace", path: "/skus/{id}/price", value: 29.99 }       ║
║                                                                        ║
║    SKU stock changed:                                                  ║
║    => { op: "replace", path: "/skus/{id}/stock", value: 50 }          ║
║                                                                        ║
║    SKU code changed:                                                   ║
║    => { op: "replace", path: "/skus/{id}/sku_code", value: "NEW-01" } ║
║                                                                        ║
║    SKU activation toggled:                                             ║
║    => { op: "replace", path: "/skus/{id}/is_active", value: false }   ║
║                                                                        ║
║  Regional Prices (nested under SKUs):                                  ║
║    Regional price added to SKU:                                        ║
║    => { op: "add", path: "/skus/{id}/regional_prices/-",              ║
║         value: { region_id: 3, original_price: 149.00, stock: 50 } }  ║
║                                                                        ║
║    Regional price removed from SKU:                                    ║
║    => { op: "remove", path: "/skus/{id}/regional_prices/{id}" }       ║
║                                                                        ║
║    Regional price fields changed:                                      ║
║    => { op: "replace",                                                 ║
║         path: "/skus/{id}/regional_prices/{id}/original_price",        ║
║         value: 199.00 }                                                ║
║    => { op: "replace",                                                 ║
║         path: "/skus/{id}/regional_prices/{id}/discount_price",        ║
║         value: 149.00 }    (or null to remove discount)                ║
║    => { op: "replace",                                                 ║
║         path: "/skus/{id}/regional_prices/{id}/stock", value: 25 }    ║
║    => { op: "replace",                                                 ║
║         path: "/skus/{id}/regional_prices/{id}/is_active",             ║
║         value: false }                                                 ║
║    Constraint: discount_price must not exceed original_price.          ║
║                                                                        ║
║  Gallery:                                                              ║
║    Gallery item added:                                                 ║
║    => { op: "add", path: "/gallery/-", value: { url: "https://...",   ║
║         position: 5, type: "image", asset_id: "uuid" } }              ║
║    (Video: add thumbnail_url + duration, set is_primary: false)        ║
║                                                                        ║
║    Gallery item removed:                                               ║
║    => { op: "remove", path: "/gallery/{id}" }                         ║
║                                                                        ║
║    Gallery reordered:                                                  ║
║    => { op: "replace", path: "/gallery/{id}/position", value: 2 }     ║
║                                                                        ║
║    Gallery primary changed:                                            ║
║    => { op: "replace", path: "/gallery/{old}/is_primary", value: 0 }  ║
║    => { op: "replace", path: "/gallery/{new}/is_primary", value: 1 }  ║
║                                                                        ║
║    Gallery type changed (image -> video):                              ║
║    Must set thumbnail_url BEFORE type change. Cannot change type on    ║
║    primary image (reassign primary first). Operations must be ordered: ║
║    1. Unset primary if needed                                          ║
║    2. Set thumbnail_url                                                ║
║    3. Replace type to "video"                                          ║
║    4. Optionally set duration                                          ║
║                                                                        ║
║    Gallery asset_id changed:                                           ║
║    => { op: "replace", path: "/gallery/{id}/asset_id",                ║
║         value: "new-uuid" }                                            ║
║                                                                        ║
║  Specifications:                                                       ║
║    Spec added / removed / reordered: same pattern as other arrays.     ║
║                                                                        ║
║    Spec type changed (text -> select or vice versa):                   ║
║    => { op: "replace", path: "/specifications/{id}/type",              ║
║         value: "select" }                                              ║
║    If type=select, value must be in the category schema options.       ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Patch Document Examples

#### Simple Field Updates

```
User changes title and description:

PATCH /seller-products/42
Content-Type: application/json-patch+json
If-Match: "a1b2c3d4e5f6789012345678abcdef01"

[
  { "op": "replace", "path": "/title", "value": "Updated Mouse Title" },
  { "op": "replace", "path": "/description", "value": "<p>New desc</p>" }
]
```

#### Adding a New Property Value + New SKU (with _tempId)

```
User adds "Red" color to an existing Color property,
then adds a new SKU for "Red, 800 DPI":

[
  {
    "op": "add",
    "path": "/properties/101/values/-",
    "value": {
      "value": "Red",
      "image_url": "https://cdn.../red.jpg",
      "position": 2,
      "_tempId": "temp-red-001"
    }
  },
  {
    "op": "add",
    "path": "/skus/-",
    "value": {
      "price": 24.99,
      "stock": 100,
      "sku_code": "RD-800",
      "is_active": true,
      "property_value_ids": [205],
      "property_value_refs": ["temp-red-001"]
    }
  }
]

The _tempId mechanism:
1. Client assigns a temporary ID string "temp-red-001" to the new Red value
2. The new SKU uses property_value_ids for existing DB values (e.g., 205
   = "800 DPI") and property_value_refs for temp references to values
   being created in the same patch
3. Server processes ops in order: creates the value, maps
   "temp-red-001" -> real BIGINT ID, merges refs into value IDs
4. Client never needs to make two separate API calls
5. _tempId is stripped from persisted data — never stored or returned
```

#### Removing a Property (cascades)

```
User removes the "DPI" property entirely:

[
  { "op": "remove", "path": "/properties/102" }
]

Server-side cascade:
- All values under property 102 are deleted
- All SKU associations referencing those values are removed
- Orphaned SKUs (with incomplete associations) are flagged as errors
- Post-operation validation checks for orphaned SKUs

If SKUs would be orphaned, the server returns 422:
"SKU '{id}' has incomplete property associations after value removal."

The frontend should warn the user before removing a property:
"Removing this property will delete X SKUs that use it."
```

#### Gallery Reorder via Drag and Drop

```
User drags gallery image from position 3 to position 0:

Before: [img-A(pos:0), img-B(pos:1), img-C(pos:2), img-D(pos:3)]
After:  [img-D(pos:0), img-A(pos:1), img-B(pos:2), img-C(pos:3)]

Generated patch:
[
  { "op": "replace", "path": "/gallery/401/position", "value": 0 },
  { "op": "replace", "path": "/gallery/398/position", "value": 1 },
  { "op": "replace", "path": "/gallery/399/position", "value": 2 },
  { "op": "replace", "path": "/gallery/400/position", "value": 3 }
]

Position values are simple integers. The frontend sends replace ops for
every item whose position changed after the drag.
```

#### Specification Reorder via Drag and Drop

```
User drags "Weight" spec from position 3 to position 1:

Before: [Brand(0), Connectivity(1), Sensor(2), Weight(3)]
After:  [Brand(0), Weight(1), Connectivity(2), Sensor(3)]

Generated patch:
[
  { "op": "replace", "path": "/specifications/504/position", "value": 1 },
  { "op": "replace", "path": "/specifications/502/position", "value": 2 },
  { "op": "replace", "path": "/specifications/503/position", "value": 3 }
]
```

#### Status Transition

```
User clicks [Submit for Review] on a draft product:

[
  { "op": "replace", "path": "/status", "value": "pending" }
]

This can be combined with other changes in the same patch.

Valid transitions (from seller perspective):
  draft     -> pending    (submit for review)
  active    -> inactive   (deactivate)
  inactive  -> active     (reactivate)
  violation -> pending    (fix and resubmit)
  any       -> deleted    (soft delete)

Admin-only transitions (NOT available to sellers via this endpoint):
  pending   -> active     (approval)
  pending   -> violation  (flagging)

Invalid transition error:
  "Cannot transition from '{from}' to '{to}'."

Completeness requirement for draft -> pending:
  The server validates that all required category schema attributes
  are present when transitioning to pending. If required variation
  attributes or specifications are missing, the server returns 422:
  "The required property '{name}' is missing."
  "The required specification '{name}' is missing."
  The frontend should validate this before submitting.
```

---

## ETag Conflict Resolution

### Happy Path (No Conflict)

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: User loads product (ETag: "abc123")                          ║
║  STEP 2: User makes changes                                           ║
║  STEP 3: User clicks [Save Changes]                                   ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  PATCH /seller-products/42                                             ║
║  Content-Type: application/json-patch+json                             ║
║  If-Match: "abc123"                                                    ║
║  Body: [ ...patch operations ]                                         ║
║                                                                        ║
║  Server checks: stored ETag == "abc123"? YES                           ║
║                                                                        ║
║  Response (200):                                                       ║
║  ETag: "def456"   <-- new ETag after update                            ║
║  {                                                                     ║
║    "message": "Product updated successfully.",                         ║
║    "data": { ...updated product with new etag }                        ║
║  }                                                                     ║
║                                                                        ║
║  Frontend:                                                             ║
║  - Update stored ETag to "def456"                                      ║
║  - Reset baseline to new product state                                 ║
║  - Toast: "Product saved"                                              ║
║  - Invalidate product list query cache                                 ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Conflict (412 Precondition Failed)

```
╔════════════════════════════════════════════════════════════════════════╗
║  SCENARIO: Two users editing the same product                         ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Timeline:                                                             ║
║  T0: User A loads product (ETag: "abc123")                             ║
║  T1: User B loads product (ETag: "abc123")                             ║
║  T2: User B saves changes -> ETag now "def456"                         ║
║  T3: User A tries to save -> sends If-Match: "abc123"                  ║
║      Server says: "abc123" != "def456" -> 412 Precondition Failed      ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
                                    │
                                    ▼
╔════════════════════════════════════════════════════════════════════════╗
║  CONFLICT RESOLUTION UI                                               ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Server returns 412:                                                   ║
║  {                                                                     ║
║    "message": "Precondition failed. The resource has been modified.    ║
║               Re-fetch and retry."                                     ║
║  }                                                                     ║
║                                                                        ║
║  Frontend shows conflict dialog:                                       ║
║                                                                        ║
║  ╔══════════════════════════════════════════════════════════════╗     ║
║  ║  Save Conflict                                      [x]     ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                                              ║     ║
║  ║  This product was modified by another user while you were    ║     ║
║  ║  editing it. Your changes could not be saved.                ║     ║
║  ║                                                              ║     ║
║  ║  Choose how to proceed:                                      ║     ║
║  ║                                                              ║     ║
║  ║  ┌──────────────────────────────────────────────────────┐   ║     ║
║  ║  │  Reload and lose my changes                          │   ║     ║
║  ║  │  Fetches the latest version. Your unsaved changes    │   ║     ║
║  ║  │  will be lost.                                       │   ║     ║
║  ║  │                                    [Reload Product]  │   ║     ║
║  ║  └──────────────────────────────────────────────────────┘   ║     ║
║  ║                                                              ║     ║
║  ║  ┌──────────────────────────────────────────────────────┐   ║     ║
║  ║  │  Force save (overwrite)                              │   ║     ║
║  ║  │  Re-fetch latest ETag and re-send your changes.      │   ║     ║
║  ║  │  This overwrites the other user's modifications.     │   ║     ║
║  ║  │                                    [Force Save]      │   ║     ║
║  ║  └──────────────────────────────────────────────────────┘   ║     ║
║  ║                                                              ║     ║
║  ╚══════════════════════════════════════════════════════════════╝     ║
║                                                                        ║
║  [Reload Product] flow:                                                ║
║  1. Re-fetch product via GET /seller-products/{id}?expand[]=...        ║
║  2. Replace all form fields with fresh data                            ║
║  3. Update stored ETag                                                 ║
║  4. Reset baseline                                                     ║
║  5. User's unsaved changes are gone                                    ║
║                                                                        ║
║  [Force Save] flow:                                                    ║
║  1. Re-fetch product to get new ETag only                              ║
║  2. Re-send PATCH with new ETag in If-Match                            ║
║  3. If 200 -> success, update baseline                                 ║
║  4. If 412 again -> repeat conflict flow (someone else saved again)    ║
║                                                                        ║
║  ⚠️ Force Save risk: The patch operations were generated by diffing    ║
║  the user's form state against the ORIGINAL baseline (before the       ║
║  other user's changes). The new ETag validates against a different      ║
║  server state. This means:                                             ║
║  - "replace" ops overwrite the other user's values                     ║
║  - "remove" ops may target IDs that no longer exist (→ 422)            ║
║  - "add" ops are generally safe (append to collections)                ║
║  If Force Save returns 422, show the errors and let the user Reload.   ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Detailed Conflict Dialog — Data Sources

```
╔════════════════════════════════════════════════════════════════════════╗
║  HOW THE CONFLICT DETAILS VIEW IS POPULATED                           ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  The 412 response itself contains ONLY:                               ║
║  { "message": "Precondition failed. The resource has been modified.   ║
║                Re-fetch and retry." }                                  ║
║                                                                        ║
║  It does NOT include the other user's changes or the current state.    ║
║                                                                        ║
║  To populate the conflict dialog, the frontend follows this sequence:  ║
║                                                                        ║
║  1. Receive 412 from PATCH attempt.                                    ║
║  2. Automatically re-fetch the product:                                ║
║     GET /seller-products/{id}?expand[]=properties&expand[]=skus       ║
║       &expand[]=specifications&expand[]=gallery&expand[]=category      ║
║       &expand[]=regional_prices&response_type=regular                  ║
║  3. Diff the re-fetched data against the form's ORIGINAL BASELINE     ║
║     (the data that was loaded when the edit page first opened).        ║
║     Differences = "OTHER USER's changes."                              ║
║  4. "YOUR changes" = the pending patch operations array (already       ║
║     computed from form state vs baseline before the PATCH was sent).   ║
║  5. Display both change summaries in the dialog.                       ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

```
╔════════════════════════════════════════════════════════════════════════╗
║  CONFLICT DETAILS UI                                                  ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ╔══════════════════════════════════════════════════════════════╗     ║
║  ║  Save Conflict — Changes Comparison              [x]        ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                                              ║     ║
║  ║  YOUR changes (unsaved):                                     ║     ║
║  ║  ┌────────────────────────────────────────────────────────┐ ║     ║
║  ║  │  · Title changed                                       │ ║     ║
║  ║  │  · 2 new SKUs added                                    │ ║     ║
║  ║  │  · BD pricing updated (+50 stock)                      │ ║     ║
║  ║  └────────────────────────────────────────────────────────┘ ║     ║
║  ║  (Source: pending patch operations array from form state)    ║     ║
║  ║                                                              ║     ║
║  ║  OTHER USER's changes (saved):                               ║     ║
║  ║  ┌────────────────────────────────────────────────────────┐ ║     ║
║  ║  │  · Status: draft -> pending                            │ ║     ║
║  ║  │  · 3 specifications updated                            │ ║     ║
║  ║  └────────────────────────────────────────────────────────┘ ║     ║
║  ║  (Source: diff of re-fetched server state vs original       ║     ║
║  ║   baseline from page load)                                  ║     ║
║  ║                                                              ║     ║
║  ║  Choose how to proceed:                                      ║     ║
║  ║                                                              ║     ║
║  ║  o Reload latest version (lose my changes)                   ║     ║
║  ║    Fetches fresh data. Your unsaved edits are discarded.     ║     ║
║  ║                                                              ║     ║
║  ║  o Force save (overwrite their changes)                      ║     ║
║  ║    ┌──────────────────────────────────────────────────────┐ ║     ║
║  ║    │  Warning: This will overwrite the other user's        │ ║     ║
║  ║    │  changes. Their status change and spec updates will   │ ║     ║
║  ║    │  be lost.                                             │ ║     ║
║  ║    └──────────────────────────────────────────────────────┘ ║     ║
║  ║    (orange warning box, only visible when this option selected)     ║
║  ║                                                              ║     ║
║  ║  o Download my changes as JSON (emergency backup)            ║     ║
║  ║    Save your pending patch operations to a file, then        ║     ║
║  ║    reload and re-apply manually.                             ║     ║
║  ║                                                              ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                          [Cancel]    [Apply Choice]         ║     ║
║  ╚══════════════════════════════════════════════════════════════╝     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Download Changes as JSON

```
╔════════════════════════════════════════════════════════════════════════╗
║  DOWNLOAD CHANGES — EMERGENCY BACKUP                                  ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  When the user selects "Download my changes as JSON" and clicks        ║
║  [Apply Choice], the browser downloads a JSON file containing the      ║
║  pending patch operations array.                                       ║
║                                                                        ║
║  File format: JSON array of RFC 6902 patch operations.                 ║
║  Example content:                                                      ║
║  [                                                                     ║
║    { "op": "replace", "path": "/title", "value": "Updated Title" },   ║
║    { "op": "add", "path": "/skus/-", "value": { ... } }               ║
║  ]                                                                     ║
║                                                                        ║
║  Filename convention:                                                  ║
║  product-{id}-changes-{ISO-timestamp}.json                             ║
║  Example: product-42-changes-2026-03-01T14-30-00.json                  ║
║                                                                        ║
║  This is a safety net, NOT a primary workflow:                          ║
║  - There is no automated re-import mechanism.                          ║
║  - The user must manually inspect the JSON and re-apply changes        ║
║    after reloading the latest product state.                           ║
║  - Patch paths reference database IDs that may have changed after      ║
║    the other user's modifications, so blind re-application may fail.   ║
║                                                                        ║
║  After download, the dialog stays open. The user typically clicks      ║
║  [Reload Product] next to get the latest state.                        ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Error Handling

### Missing If-Match Header (428)

```
Should never happen (frontend always sends it), but if it does:

Response (428):
{ "message": "If-Match header is required for PATCH operations." }

Frontend: Show error toast. This is a programming error.
```

### Wrong Content-Type (415)

```
Response (415):
{ "message": "Unsupported media type. Expected: application/json-patch+json." }

Frontend: Show error toast. Programming error in request setup.
```

### Test Operation Failed (409)

```
If the patch includes a "test" operation that fails:

[
  { "op": "test", "path": "/title", "value": "Expected Title" },
  { "op": "replace", "path": "/title", "value": "New Title" }
]

If current title != "Expected Title":
Response (409):
{ "message": "Test failed at '/title': expected 'Expected Title', got 'Actual Title'." }

The test operation is used for safety checks. The frontend can include
test ops to verify assumptions before applying changes.
```

### Validation Errors (422)

Same handling as create flow. Server validates the final state after applying all patch operations. Errors are mapped to form fields.

```
Example: User removes a required property via patch.
Server applies the remove, then runs post-op validation.

Response (422):
{
  "message": "The given data was invalid.",
  "errors": {
    "properties": ["The required property 'Color' is missing."]
  }
}
```

### Visual Error Display Patterns

```
╔════════════════════════════════════════════════════════════════════════╗
║  ERROR DISPLAY BY LOCATION                                            ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  1. Single field error (e.g., title):                                  ║
║  ┌─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┐            ║
║  │  Product Title *                                       │            ║
║  │  ┌───────────────────────────────────────────────┐    │            ║
║  │  │                                            ⚠️  │    │            ║
║  │  └───────────────────────────────────────────────┘    │            ║
║     ⚠️ The title field is required.                                    ║
║  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┘            ║
║  (red left-border on field group, ⚠️ icon in input, red msg below)     ║
║                                                                        ║
║  2. Nested path error (properties.0.name):                             ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  VARIATIONS section highlighted with red left-border              │║
║  │  ┌────────────────────────────────────────────────────────────┐  │║
║  │  │  Property 1: [___________]  ← red border, ⚠️ icon         │  │║
║  │  │  ⚠️ Not defined in category schema.                        │  │║
║  │  └────────────────────────────────────────────────────────────┘  │║
║  └──────────────────────────────────────────────────────────────────┘║
║  (parent section gets subtle red-50 bg, specific field shows error)    ║
║                                                                        ║
║  3. Array element error (skus.2.price):                                ║
║  ┌───┬───────┬──────┬─────────┬───────┐                              ║
║  │ = │ Black │ S    │ 24.99   │ 100   │  (normal row)                ║
║  │ = │ Black │ M    │ 24.99   │ 100   │  (normal row)                ║
║  │ = │ White │ S    │ [0.00]  │ 50    │  ← red border on price cell  ║
║  │   │       │      │ ⚠️ Min   │       │                              ║
║  │   │       │      │  0.01   │       │                              ║
║  └───┴───────┴──────┴─────────┴───────┘                              ║
║  (SKU row highlighted with red-50 bg, specific cell has red border)    ║
║                                                                        ║
║  4. Root array error (gallery):                                        ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  ⚠️ Exactly one gallery image must be marked as primary.          │║
║  └──────────────────────────────────────────────────────────────────┘║
║  (red banner above the gallery section, red-50 bg)                     ║
║                                                                        ║
║  5. Error summary sticky bar (appears at top when errors exist):       ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  ⚠️ 3 validation errors                     [Scroll to first ▲] │║
║  └──────────────────────────────────────────────────────────────────┘║
║  (sticky below header, red-50 bg, clicking scrolls to first error)     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Product Not Found (404)

```
When loading the edit page OR during PATCH:

Response (404):
{ "message": "Product not found." }

On page load:
  Frontend redirects to /products with toast: "Product not found."
  This happens when the ID does not exist or belongs to another company.

During PATCH:
  Product was deleted while the user was editing.
  Frontend: toast "This product no longer exists." + redirect to /products.
```

### Category Change Re-validation

```
When shipping_category_id is changed via PATCH, the server re-validates all
properties and specifications against the NEW category's schema:

[
  { "op": "replace", "path": "/shipping_category_id", "value": "new-cat-uuid" },
  { "op": "add", "path": "/properties/-", "value": { ... new props ... } }
]

If existing properties/specs don't match the new schema:
Response (422):
{
  "message": "The given data was invalid.",
  "errors": {
    "properties.0.name": [
      "Property/specification 'DPI' does not conform to the new category schema."
    ]
  }
}

The frontend should warn the user when they change categories that existing
properties and specifications may need to be updated.
```

### Full PATCH Error Reference

```
╔════════════════════════════════════════════════════════════════════════╗
║  ALL PATCH ENDPOINT ERRORS                                            ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  404  Product not found.                                               ║
║  409  Test failed at '{path}': expected {expected}, got {actual}.       ║
║  412  Precondition failed. The resource has been modified.             ║
║       Re-fetch and retry.                                              ║
║  415  Unsupported media type. Expected: application/json-patch+json.   ║
║  428  If-Match header is required for PATCH operations.                ║
║  400  Failed to update product. Please try again.                      ║
║                                                                        ║
║  422 — Structural:                                                     ║
║    Patch document must be a JSON array of operations.                  ║
║    Operation at index {i} missing required field: {field}.             ║
║    Operation at index {i}: unsupported op '{op}'.                      ║
║    Path '{path}' is not a valid target for this resource.              ║
║                                                                        ║
║  422 — Target resolution:                                              ║
║    Cannot replace '{path}' -- ID not found.                            ║
║    Cannot remove '{path}' -- ID not found.                             ║
║    ID {id} not found in {collection}.                                  ║
║                                                                        ║
║  422 — Value validation:                                               ║
║    Title must be a string between 1 and 255 characters.                ║
║    Status must be one of: draft, pending, active, inactive,            ║
║      violation, deleted.                                               ║
║    Cannot transition from '{from}' to '{to}'.                          ║
║    SKU price must be at least 0.01.                                    ║
║    SKU stock must be a non-negative integer.                           ║
║    Invalid value at '{path}': expected {type}.                         ║
║                                                                        ║
║  422 — Add operations:                                                 ║
║    Property must include name, position, and values.                   ║
║    SKU must include price, stock, and property_value_ids.              ║
║    Gallery item must include url and position.                         ║
║    Gallery video must include a thumbnail_url.                         ║
║    Only images can be marked as primary.                               ║
║    Gallery type must be one of: image, video.                          ║
║    Specification must include name, value, and position.               ║
║    Regional price must include region_id, original_price, and stock.   ║
║                                                                        ║
║  422 — Gallery type change:                                            ║
║    Cannot change type to video without a thumbnail_url.                ║
║      Set thumbnail_url first.                                          ║
║    Cannot change type to video on primary gallery item.                ║
║      Reassign primary first.                                           ║
║    Gallery thumbnail URL must be a valid URL (max 500 chars) or null.  ║
║    Gallery duration must be a non-negative integer or null.            ║
║                                                                        ║
║  422 — TempId:                                                         ║
║    The temporary reference '{ref}' was not found in any add operation.  ║
║    The SKU references property values that do not exist: {ids}.        ║
║                                                                        ║
║  422 — Regional pricing:                                               ║
║    Region ID {id} does not exist.                                      ║
║    Region ID {id} is already assigned to this SKU.                     ║
║    Regional original price must be at least 0.01.                      ║
║    Regional discount price must be at least 0.01 or null.              ║
║    Regional stock must be a non-negative integer.                      ║
║                                                                        ║
║  422 — Post-operation constraints:                                     ║
║    Product cannot have more than 3 properties. Current: {n}.           ║
║    Property '{name}' cannot have more than 50 values. Current: {n}.    ║
║    Product cannot have more than 100 SKUs. Current: {n}.               ║
║    Product cannot have more than 50 specifications. Current: {n}.      ║
║    Product cannot have more than 20 gallery items. Current: {n}.       ║
║    Exactly one gallery image must be marked as primary.                ║
║    Duplicate property name: '{name}'.                                   ║
║    Duplicate specification name: '{name}'.                              ║
║    The sku_code '{code}' is already used by another SKU.               ║
║    Duplicate value '{value}' in property '{name}'.                      ║
║    SKU '{id}' has incomplete property associations after               ║
║      value removal.                                                    ║
║    Property/specification '{name}' does not conform to the new         ║
║      category schema.                                                  ║
║    Gallery item '{id}' is a video but has is_primary=true.             ║
║                                                                        ║
║  Field path mapping (server -> form):                                  ║
║    "properties.0.name"         -> properties[0].name                   ║
║    "skus.2.price"              -> skus[2].price                        ║
║    "gallery"                   -> root gallery section                  ║
║    "specifications.0.value"    -> specifications[0].value              ║
║    "skus.1.regional_prices.0"  -> skus[1].regional_prices[0]          ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Transaction Failure (400)

```
Response (400):
{ "message": "Failed to update product. Please try again." }

Frontend: Toast error. User retries.
```

---

## Unsaved Changes Warning

```
╔════════════════════════════════════════════════════════════════════════╗
║  User tries to navigate away with unsaved changes                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ╔══════════════════════════════════════════════════════════════╗     ║
║  ║  Unsaved Changes                                    [x]     ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                                              ║     ║
║  ║  You have unsaved changes. Are you sure you want to          ║     ║
║  ║  leave this page?                                            ║     ║
║  ║                                                              ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║          [Stay on Page]         [Leave Without Saving]      ║     ║
║  ╚══════════════════════════════════════════════════════════════╝     ║
║                                                                        ║
║  Two dialog mechanisms depending on navigation type:                    ║
║                                                                        ║
║  Browser beforeunload (native dialog, cannot be styled):               ║
║  - Tab close / browser close                                           ║
║  - Browser back to external page (outside app)                         ║
║  - URL bar navigation to a different site                              ║
║  The browser shows its own generic "Leave site?" dialog per spec.      ║
║  Custom text is ignored by modern browsers.                            ║
║                                                                        ║
║  In-app navigation (custom styled dialog as shown above):              ║
║  - Clicking a sidebar link                                             ║
║  - Clicking [Cancel] button                                            ║
║  - In-app router.push / router.back (Next.js router intercept)        ║
║  Uses Next.js router events to intercept and show the custom dialog.   ║
║                                                                        ║
║  Detection: React Hook Form's formState.isDirty                        ║
║  Implementation: Next.js router event + beforeunload listener          ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Form Dirty State Indicator

```
╔════════════════════════════════════════════════════════════════════════╗
║  STICKY SAVE STATUS BAR (below page header)                           ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  CLEAN (all saved):                                                    ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  ✓ All changes saved                                   ETag: a1b│║
║  └──────────────────────────────────────────────────────────────────┘║
║  (green text, ✓ icon, subtle green-50 bg)                             ║
║                                                                        ║
║  DIRTY (unsaved edits):                                                ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  ● Unsaved changes (6 fields changed)              [Save] [Undo]│║
║  └──────────────────────────────────────────────────────────────────┘║
║  (orange text, ● dot with subtle pulse animation, orange-50 bg)        ║
║  Ctrl/Cmd+S keyboard shortcut triggers [Save]                          ║
║                                                                        ║
║  Hovering "6 fields changed" shows a tooltip listing which sections    ║
║  have changes:                                                         ║
║  ┌──────────────────────────────────────────────────────────┐        ║
║  │  Basic Info: title, description                          │        ║
║  │  SKUs: 2 prices changed                                  │        ║
║  │  Gallery: 1 image added                                  │        ║
║  └──────────────────────────────────────────────────────────┘        ║
║  The count and breakdown are computed by comparing current form        ║
║  state against the baseline, grouped by section.                       ║
║                                                                        ║
║  SAVING (mutation in progress):                                        ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  ⏳ Saving changes...                                             │║
║  └──────────────────────────────────────────────────────────────────┘║
║  (blue text, spinner icon, blue-50 bg)                                 ║
║                                                                        ║
║  ERROR (save failed):                                                  ║
║  ┌──────────────────────────────────────────────────────────────────┐║
║  │  ⚠️ Save failed — connection error               [Retry] [Undo] │║
║  └──────────────────────────────────────────────────────────────────┘║
║  (red text, ⚠️ icon, red-50 bg, [Retry] re-sends the same PATCH)     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## PATCH API Request Shape

```
PATCH /api/seller/agent/seller-product/v1/seller-products/{id}

Headers:
  Content-Type: application/json-patch+json
  If-Match: "a1b2c3d4e5f6789012345678abcdef01"

Body (JSON array of RFC 6902 operations):
[
  { "op": "replace", "path": "/title", "value": "New Title" },
  { "op": "replace", "path": "/description", "value": "<p>New</p>" },
  { "op": "add", "path": "/specifications/-", "value": {
      "name": "Weight", "value": "85g", "type": "text", "position": 5
    }
  },
  { "op": "remove", "path": "/gallery/405" },
  { "op": "replace", "path": "/skus/301/price", "value": 34.99 }
]

Supported operations: add, remove, replace, test
NOT supported: move, copy

Path addressing uses database IDs (BIGINT), not array indices:
  /properties/101        <- property with id=101
  /properties/101/values/201  <- value with id=201
  /skus/301              <- SKU with id=301
  /gallery/401           <- gallery item with id=401

All addressable paths:

  Top-level fields:
    /title                                          replace   string (1-255)
    /description                                    replace   string (max 65535)
    /status                                         replace   string (validated transition)
    /shipping_category_id                           replace   string (leaf UUID)

  Properties:
    /properties/-                                   add       { name, position, values[] }
    /properties/{id}                                remove
    /properties/{id}/name                           replace   string (max 100)
    /properties/{id}/position                       replace   integer
    /properties/{id}/values/-                       add       { value, image_url?, position, _tempId? }
    /properties/{id}/values/{id}                    remove
    /properties/{id}/values/{id}/value              replace   string (max 100)
    /properties/{id}/values/{id}/image_url          replace   string (url, max 500)
    /properties/{id}/values/{id}/position           replace   integer

  SKUs:
    /skus/-                                         add       { price, stock, sku_code?, is_active?,
                                                                property_value_ids[], property_value_refs[]? }
    /skus/{id}                                      remove
    /skus/{id}/price                                replace   number (min 0.01, max 9999999999.99)
    /skus/{id}/stock                                replace   integer (min 0)
    /skus/{id}/sku_code                             replace   string (max 50, unique per product)
    /skus/{id}/is_active                            replace   boolean

  SKU Regional Prices:
    /skus/{id}/regional_prices/-                    add       { region_id, original_price, stock,
                                                                discount_price?, is_active? }
    /skus/{id}/regional_prices/{id}                 remove
    /skus/{id}/regional_prices/{id}/original_price  replace   number (min 0.01)
    /skus/{id}/regional_prices/{id}/discount_price  replace   number (min 0.01) or null
    /skus/{id}/regional_prices/{id}/stock           replace   integer (min 0)
    /skus/{id}/regional_prices/{id}/is_active       replace   boolean

  Specifications:
    /specifications/-                               add       { name, value, type?, position }
    /specifications/{id}                            remove
    /specifications/{id}/name                       replace   string (max 100)
    /specifications/{id}/value                      replace   string (max 500)
    /specifications/{id}/type                       replace   string (text or select)
    /specifications/{id}/position                   replace   integer

  Gallery:
    /gallery/-                                      add       { url, position, asset_id?, type?,
                                                                thumbnail_url?, duration?, is_primary? }
    /gallery/{id}                                   remove
    /gallery/{id}/url                               replace   string (url, max 500)
    /gallery/{id}/asset_id                          replace   string (uuid) or null
    /gallery/{id}/type                              replace   string (image or video)
    /gallery/{id}/thumbnail_url                     replace   string (url, max 500) or null
    /gallery/{id}/duration                          replace   integer (min 0) or null
    /gallery/{id}/position                          replace   integer
    /gallery/{id}/is_primary                        replace   boolean (type=image only)

  "test" operation is valid on ALL addressable paths above.
  Aborts entire patch with 409 if value does not match current state.

Post-operation constraints (checked AFTER all ops execute):
  - Max 3 properties per product
  - Max 50 values per property
  - Max 100 SKUs per product
  - Max 50 specifications per product
  - Max 20 gallery items (images + videos combined)
  - Exactly 1 gallery image must be marked as primary
  - No duplicate property names per product
  - No duplicate spec names per product
  - No duplicate sku_code values per product (where not null)
  - No duplicate property values within same property
  - No duplicate region_id per SKU
  - discount_price must not exceed original_price (regional pricing)
  - Videos cannot be marked as primary
  - Orphaned SKUs (incomplete property associations after value removal) are errors

Response (200):
  Headers: ETag: "new-etag-value"
  Body: { "message": "Product updated successfully.",
          "data": { "object": "SellerProduct", ...full product with all relations } }
```

---

## TanStack Query Integration

```
Query key for edit page (no region_id -- edit loads ALL regions):
["seller-products", "detail", id, { expand: [...] }]

Fetching:
useQuery({
  queryKey: ['seller-products', 'detail', id, {
    expand: ['properties', 'skus', 'specifications', 'gallery', 'category', 'regional_prices']
  }],
  queryFn: () => api.product.get(`/seller-products/${id}`, {
    params: { 'expand[]': [...], response_type: 'regular' }
  }),
  // Retry on 404 is pointless -- product does not exist
  retry: (failureCount, error) => error.status !== 404,
})

Mutation:
useMutation({
  mutationFn: ({ id, patch, etag }) =>
    api.product.patch(`/seller-products/${id}`, patch, {
      headers: {
        'Content-Type': 'application/json-patch+json',
        'If-Match': `"${etag}"`
      }
    }),
  onSuccess: (response) => {
    // Response wrapper: { message: "...", data: { object: "SellerProduct", ... } }
    const product = response.data.data
    // Update local ETag (from response body -- header also has it)
    setEtag(product.etag)
    // Update baseline for change tracking
    setBaseline(product)
    // Invalidate list queries
    queryClient.invalidateQueries({ queryKey: ['seller-products', 'list'] })
    queryClient.invalidateQueries({ queryKey: ['seller-products', 'status-counts'] })
    // Update detail cache
    queryClient.setQueryData(
      ['seller-products', 'detail', id],
      response
    )
    toast.success('Product saved')
  },
  onError: (error) => {
    if (error.status === 412) {
      // ETag mismatch -- another user modified the product
      showConflictDialog()
    } else if (error.status === 422) {
      mapServerErrors(error.data.errors, form.setError)
    } else if (error.status === 409) {
      // "test" operation failed -- assumption violated
      toast.error(error.data.message)
    } else if (error.status === 428) {
      // Missing If-Match header -- programming error
      toast.error('Missing ETag header. Please reload the page.')
    } else if (error.status === 415) {
      // Wrong Content-Type -- programming error
      toast.error('Request format error. Please reload the page.')
    } else if (error.status === 404) {
      // Product was deleted while editing
      toast.error('This product no longer exists.')
      router.push('/products')
    } else if (error.status === 400) {
      // Database transaction failure
      toast.error('Save failed. Please try again.')
    }
  }
})

Delete from edit page:
  The edit page can include a [Delete] action (visible for non-deleted products).
  Two approaches:

  Option A — PATCH status to deleted:
  [{ "op": "replace", "path": "/status", "value": "deleted" }]
  Requires If-Match header. Returns 200 with updated product (status=deleted, deleted_at set).

  Option B — DELETE endpoint:
  DELETE /api/seller/agent/seller-product/v1/seller-products/{id}
  No If-Match required. Idempotent (already-deleted returns 200).
  Response: { "message": "Product deleted successfully." }

  After either: invalidate list + status-counts, redirect to /products.
```
