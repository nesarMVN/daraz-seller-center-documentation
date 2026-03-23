# Product Edit Flow — E2E Test Architecture

## Page Object

### ProductEditPage

```
File: e2e/pages/product-edit.page.ts
Class: ProductEditPage
```

**Constructor parameter:** `page: Page`

**Locators:**

| Locator | Selector | Description |
|---------|----------|-------------|
| `heading` | `page.getByRole('heading', { name: /edit product/i })` | Page heading |
| `breadcrumb` | `page.getByText('Products > Edit Product')` | Breadcrumb navigation |
| `statusBadge` | `page.getByTestId('product-status-badge')` | Current product status badge |
| `etagDisplay` | `page.getByTestId('etag-display')` | ETag value display |
| `createdAt` | `page.getByText(/created:/i)` | Created date display |
| `sourceInfo` | `page.getByTestId('source-product-info')` | Source product link/info |
| `categorySection` | `page.getByRole('region', { name: /category/i })` | Category section |
| `basicInfoSection` | `page.getByRole('region', { name: /basic info/i })` | Basic info section |
| `variationsSection` | `page.getByRole('region', { name: /variations/i })` | Variations section |
| `specificationsSection` | `page.getByRole('region', { name: /specifications/i })` | Specifications section |
| `gallerySection` | `page.getByRole('region', { name: /gallery/i })` | Gallery section |
| `regionalPricingSection` | `page.getByRole('region', { name: /regional pricing/i })` | Regional pricing section |
| `actionBar` | `page.getByTestId('action-bar')` | Sticky bottom action bar |
| `cancelButton` | `actionBar.getByRole('button', { name: /cancel/i })` | Cancel button |
| `saveDraftButton` | `actionBar.getByRole('button', { name: /save.*draft/i })` | Save draft button (draft status only) |
| `saveChangesButton` | `actionBar.getByRole('button', { name: /save changes/i })` | Save changes button |
| `submitButton` | `actionBar.getByRole('button', { name: /submit.*review/i })` | Submit for review (draft only) |
| `resubmitButton` | `actionBar.getByRole('button', { name: /resubmit/i })` | Resubmit button (violation only) |
| `deactivateButton` | `actionBar.getByRole('button', { name: /deactivate/i })` | Deactivate button (active only) |
| `activateButton` | `actionBar.getByRole('button', { name: /activate/i })` | Activate button (inactive only) |
| `titleInput` | `page.getByLabel(/product title/i)` | Title input |
| `descriptionEditor` | `page.getByTestId('description-editor')` | Rich text editor |
| `dirtyIndicator` | `page.getByTestId('save-status-bar')` | Save status bar (clean/dirty/saving/error) |
| `unsavedChangesDialog` | `page.getByRole('alertdialog', { name: /unsaved changes/i })` | Unsaved changes warning |
| `stayButton` | `unsavedChangesDialog.getByRole('button', { name: /stay/i })` | Stay on page button |
| `leaveButton` | `unsavedChangesDialog.getByRole('button', { name: /leave/i })` | Leave without saving button |
| `conflictDialog` | `page.getByRole('dialog', { name: /save conflict/i })` | ETag conflict dialog |
| `reloadButton` | `conflictDialog.getByRole('button', { name: /reload/i })` | Reload product button |
| `forceSaveButton` | `conflictDialog.getByRole('button', { name: /force save/i })` | Force save button |
| `downloadChangesButton` | `conflictDialog.getByRole('button', { name: /download/i })` | Download changes as JSON |
| `violationBanner` | `page.getByTestId('violation-banner')` | Violation status banner |
| `deletedBanner` | `page.getByTestId('deleted-banner')` | Deleted product read-only banner |
| `loadingSkeleton` | `page.getByTestId('edit-form-skeleton')` | Loading skeleton |
| `toast` | `page.getByRole('status')` | Toast notification |
| `errorSummary` | `page.getByTestId('error-summary-bar')` | Error summary bar |

**Actions:**

| Method | Description |
|--------|-------------|
| `async goto(productId: number)` | Navigates to `/products/{id}/edit` |
| `async fillTitle(title: string)` | Clears and types new title |
| `async fillDescription(html: string)` | Clears and types new description |
| `async clickSaveChanges()` | Clicks save changes button |
| `async clickSaveDraft()` | Clicks save draft button |
| `async clickSubmitForReview()` | Clicks submit for review |
| `async clickResubmit()` | Clicks resubmit (violation) |
| `async clickCancel()` | Clicks cancel button |
| `async clickDeactivate()` | Clicks deactivate button |
| `async clickActivate()` | Clicks activate button |
| `async confirmLeave()` | Clicks leave in unsaved changes dialog |
| `async stayOnPage()` | Clicks stay in unsaved changes dialog |
| `async reloadFromConflict()` | Clicks reload in conflict dialog |
| `async forceSave()` | Clicks force save in conflict dialog |
| `async downloadChanges()` | Clicks download changes in conflict dialog |
| `async waitForProductLoad()` | Waits for skeleton to disappear and form to populate |
| `async waitForSaveResponse()` | Waits for PATCH response |
| `getDirtyFieldCount(): Locator` | Returns dirty field count text |

---

## Helpers

- `productApiHelper.createFullProduct(request)` — Creates a product for editing
- `productApiHelper.getProduct(request, id)` — Fetches product to verify changes
- `productApiHelper.setProductStatus(request, id, status)` — Sets product status for status-specific tests
- `productTestData` — Data generators

---

## Fixture Wiring

```typescript
productEditPage: async ({ page }, use) => {
  await use(new ProductEditPage(page))
}
```

Worker-scoped fixture for product data:
```typescript
productForEdit: [
  async ({ request }, use) => {
    const product = await productApiHelper.createFullProduct(request)
    await use(product)
    await productApiHelper.deleteProduct(request, product.id).catch(() => {})
  },
  { scope: 'worker' }
]
```

---

## Spec Files

| Spec File | Concern | Estimated Scenarios |
|-----------|---------|-------------------|
| `product-edit-page.spec.ts` | Page layout, data loading, pre-fill | 8 |
| `product-edit-changes.spec.ts` | Change tracking, dirty state | 6 |
| `product-edit-save.spec.ts` | Save workflow, JSON Patch generation | 6 |
| `product-edit-conflict.spec.ts` | ETag conflict (412) resolution | 6 |
| `product-edit-validation.spec.ts` | Edit validation errors (422) | 10 |
| `product-edit-status.spec.ts` | Status transitions from edit page | 5 |
| `product-edit-cancel.spec.ts` | Cancel and unsaved changes | 4 |
| `product-edit-violation.spec.ts` | Violation banner and resubmission | 4 |
| `product-edit-errors.spec.ts` | All PATCH error scenarios | 7 |
| `product-edit-delete.spec.ts` | Delete from edit page | 3 |

---

## Test Scenarios

### Data Loading and Pre-fill (`product-edit-page.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 1 | should show loading skeleton while fetching product data | Navigate to edit page | Observe during load | Skeleton loaders matching section layout visible. Form not interactive |
| 2 | should pre-fill title from loaded product data | Create product with title "E2E Mouse Test" | Navigate to edit | Title input has value "E2E Mouse Test" |
| 3 | should pre-fill description from loaded product data | Create product with HTML description | Navigate to edit | Rich text editor contains formatted HTML content |
| 4 | should display selected category from loaded product | Product has category set | Navigate to edit | Category field shows full path: "Electronics > ... > Mouse" |
| 5 | should pre-fill property names, values, and SKU matrix | Product has Color (Black, White) × DPI (800, 1600) | Navigate to edit | 2 properties visible with correct values. 4 SKU rows with correct prices, stocks, codes |
| 6 | should pre-fill specifications from loaded product | Product has 5 specifications | Navigate to edit | All 5 spec rows visible with correct names and values |
| 7 | should pre-fill gallery items with correct order and primary | Product has 3 images, first is primary | Navigate to edit | 3 gallery cards in correct order. First has primary badge |
| 8 | should display status badge, source info, ETag, and dates | Product loaded | Observe header area | Status badge with correct color. Created/updated dates. ETag value. Source product ID if present |

### Change Tracking and Dirty State (`product-edit-changes.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 9 | should show clean state indicator when no changes made | Load product for edit | Observe status bar | Green text: "All changes saved" |
| 10 | should show dirty state indicator when title is changed | Load product | Change title text | Orange indicator: "Unsaved changes (1 field changed)" with pulse animation |
| 11 | should show correct dirty field count as multiple fields change | Change title, change a SKU price, add a gallery image | Observe status bar | "Unsaved changes (3 fields changed)". Hover shows: "Basic Info: title", "SKUs: 1 price changed", "Gallery: 1 image added" |
| 12 | should return to clean state after successful save | Make changes, save | Observe status bar | Back to green: "All changes saved". New ETag displayed |
| 13 | should show error state in status bar when save fails | Make changes, save fails with 422 | Observe status bar | Red: "Save failed — ..." with Retry button |
| 14 | should support Ctrl/Cmd+S keyboard shortcut to save | Make changes | Press Ctrl+S (or Cmd+S on Mac) | Save triggered. Same as clicking Save Changes |

### Save Changes Workflow (`product-edit-save.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 15 | should send JSON Patch with only changed fields | Change title from "Old" to "New" | Click Save Changes | PATCH request with Content-Type: application/json-patch+json, If-Match header, body: `[{ op: "replace", path: "/title", value: "New" }]` |
| 16 | should update stored ETag after successful save | Save changes | Observe | New ETag from response stored. Next save uses new ETag in If-Match |
| 17 | should reset baseline to new product state after save | Save changes to title | Change title again, save again | Second PATCH only includes diff from new baseline, not from original load |
| 18 | should show loading state on save button during PATCH | Click Save Changes | Observe button | Button: "Saving..." with spinner. All action buttons disabled |
| 19 | should invalidate product list cache after successful save | Save changes | Navigate to product list | Product list shows updated data (e.g., new title) |
| 20 | should show success toast after save | Save changes | Observe | Toast: "Product saved" (green) |

### ETag Conflict Resolution (`product-edit-conflict.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 21 | should show conflict dialog when server returns 412 | Load product (ETag A). Modify product externally (ETag B). User saves with ETag A | Observe | Conflict dialog visible: "This product was modified by another user while you were editing it" |
| 22 | should show your changes and other user's changes in conflict dialog | Conflict dialog open | Observe dialog content | "YOUR changes" section lists pending ops. "OTHER USER's changes" section lists diffs from re-fetched data vs original baseline |
| 23 | should reload product and discard changes when Reload is clicked | Conflict dialog open | Click "Reload Product" | Product re-fetched. Form fields updated with server data. User's unsaved changes lost. ETag updated. Baseline reset |
| 24 | should re-send patch with new ETag when Force Save is clicked | Conflict dialog open | Click "Force Save" | Re-fetch product to get new ETag only. Re-send PATCH with new ETag. On 200: success. On 412 again: conflict dialog reappears |
| 25 | should download pending changes as JSON file | Conflict dialog open | Click "Download my changes" | Browser downloads JSON file: `product-{id}-changes-{timestamp}.json` containing the patch operations array |
| 26 | should handle force save returning 422 | Force save, but patch ops reference removed IDs | Observe | 422 errors shown. User advised to reload. Conflict dialog stays open |

### Edit Validation Errors (`product-edit-validation.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 27 | should show error when replacing title with empty string | Clear title | Save | Error: "Title must be a string between 1 and 255 characters" |
| 28 | should show error when replacing title exceeding 255 chars | Type 260-char title | Save | Error on title field |
| 29 | should show error for invalid status transition | Attempt draft -> active (admin-only) | Save with status replace | Error: "Cannot transition from 'draft' to 'active'" |
| 30 | should show error when adding property exceeds 3 max | Product has 3 properties | Add 4th property, save | Error: "Product cannot have more than 3 properties. Current: 4" |
| 31 | should show error when adding SKU exceeds 100 max | Product near SKU limit | Add values to exceed 100 | Error: "Product cannot have more than 100 SKUs" |
| 32 | should show error for orphaned SKU after property value removal | Product has properties and SKUs | Remove a property value that SKUs reference | Error: "SKU '{id}' has incomplete property associations after value removal" |
| 33 | should show error when adding duplicate property name | Product has "Color" property | Add another property named "Color" | Error: "Duplicate property name: 'Color'" |
| 34 | should show error when adding duplicate spec name | Product has "Brand" spec | Add another spec named "Brand" | Error: "Duplicate specification name: 'Brand'" |
| 35 | should show error when gallery exceeds 20 after adding | Product has 18 gallery items | Add 5 more | Error: "Product cannot have more than 20 gallery items. Current: 23" |
| 36 | should show error when removing primary image leaves no primary | Product has 2 images, first is primary | Remove primary image | Error: "Exactly one gallery image must be marked as primary" |

### Status Transitions (`product-edit-status.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 37 | should submit draft product for review | Create draft product | Navigate to edit, click "Submit for Review" | PATCH includes `{ op: "replace", path: "/status", value: "pending" }`. Toast: "Product submitted for review". Status badge updates to Pending |
| 38 | should deactivate active product from edit page | Create active product | Navigate to edit, click "Deactivate" | PATCH status to inactive. Toast: "Product deactivated". Status badge updates |
| 39 | should activate inactive product from edit page | Create inactive product | Navigate to edit, click "Activate" | PATCH status to active. Toast. Badge updates |
| 40 | should show correct action buttons per status | Create products in each status | Navigate to edit for each | Draft: Cancel, Save Draft, Submit. Active: Cancel, Save Changes, Deactivate. Inactive: Cancel, Save Changes, Activate. Violation: Cancel, Save Changes, Resubmit. Pending: Cancel, Save Changes. Deleted: no buttons (read-only) |
| 41 | should combine status transition with other edits in single PATCH | Draft product, change title | Click "Submit for Review" | Single PATCH containing both `replace /title` and `replace /status` ops |

### Cancel and Unsaved Changes (`product-edit-cancel.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 42 | should show unsaved changes dialog when cancel is clicked with dirty form | Make changes | Click "Cancel" | Dialog: "You have unsaved changes. Are you sure you want to leave?" with "Stay on Page" and "Leave Without Saving" |
| 43 | should stay on page when "Stay on Page" is clicked | Unsaved changes dialog visible | Click "Stay on Page" | Dialog closes. User remains on edit page. Changes preserved |
| 44 | should navigate away when "Leave Without Saving" is clicked | Unsaved changes dialog visible | Click "Leave Without Saving" | Navigate to product list. Changes discarded |
| 45 | should not show unsaved changes dialog when form is clean | No changes made | Click "Cancel" | Direct navigation to product list. No dialog |

### Violation Resubmission (`product-edit-violation.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 46 | should display violation banner with reason when status is violation | Create violation product | Navigate to edit | Red banner: "This product was flagged for a policy violation. Reason: {reason}. Fix the issues below and click [Resubmit]" |
| 47 | should show Resubmit button in action bar for violation products | Violation product loaded | Observe action bar | "Resubmit" button visible alongside "Cancel" and "Save Changes" |
| 48 | should resubmit product from violation to pending | Fix issues on violation product | Click "Resubmit" | PATCH with `replace /status` from "violation" to "pending" plus any other edits. Toast: "Product resubmitted for review". Banner disappears. Badge updates to Pending |
| 49 | should validate all required fields before resubmission | Violation product with missing required specs | Click "Resubmit" | Validation errors shown (same as submit-for-review validation). Product NOT resubmitted until all required fields filled |

### PATCH Error Scenarios (`product-edit-errors.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 50 | should show toast for 415 wrong Content-Type | Intercept to return 415 | Save | Toast: "Unsupported media type. Expected: application/json-patch+json" (programming error) |
| 51 | should show toast for 428 missing If-Match header | Intercept to return 428 | Save | Toast: "If-Match header is required for PATCH operations" (programming error) |
| 52 | should handle 409 test operation failure | PATCH includes test op that fails | Save | Toast with test failure message: "Test failed at '{path}': expected {expected}, got {actual}" |
| 53 | should redirect to product list on 404 during PATCH | Product deleted while editing | Save | Toast: "This product no longer exists." Redirect to /products |
| 54 | should show toast for 400 transaction failure | Server returns 400 | Save | Toast: "Save failed. Please try again." Save status bar shows error with Retry |
| 55 | should map 422 errors to form fields | Server returns 422 with field paths | Save | Errors mapped: `properties.0.name` -> property row 0 name input red border. `skus.2.price` -> SKU row 2 price cell red border. Scroll to first error |
| 56 | should show error for category change schema violation | Change category, existing props don't match new schema | Save | Error: "Property/specification '{name}' does not conform to the new category schema" |

### Delete from Edit Page (`product-edit-delete.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 57 | should delete product via DELETE endpoint from edit page | Active product loaded | Click Delete, confirm | DELETE /seller-products/{id}. Toast: "Product deleted". Redirect to /products. Product list and status counts invalidated |
| 58 | should show confirmation dialog before delete | Click Delete | Observe | Dialog: "Delete this product?" with product name. "Cancel" and "Delete" buttons |
| 59 | should show read-only state for deleted products | Navigate to edit page for deleted product | Observe | Banner: "This product has been deleted." All fields disabled. No action buttons. deleted_at timestamp shown |

### Loading States

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 60 | should redirect to product list with toast on 404 during load | Navigate to edit with non-existent ID | Observe | Redirect to /products. Toast: "Product not found" |
| 61 | should not retry on 404 | Navigate to non-existent product | Observe network | Only one GET request. No retry (query retry disabled for 404) |

### JSON Patch Generation (Verified Through API)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 62 | should generate replace ops for simple field changes | Change title and description | Save, intercept PATCH | Body contains `replace /title` and `replace /description` ops only |
| 63 | should generate add ops for new property values with _tempId | Add a new color value "Red" | Save, intercept | Body contains `add /properties/{id}/values/-` with `_tempId` field. New SKU references use `property_value_refs` |
| 64 | should generate remove ops for deleted entities | Remove a specification | Save, intercept | Body contains `remove /specifications/{id}` |
| 65 | should generate position replace ops for gallery reorder | Drag gallery items to reorder | Save, intercept | Body contains multiple `replace /gallery/{id}/position` ops |
| 66 | should generate regional price ops for add/edit/remove | Add BD pricing, edit AE pricing, remove US pricing | Save, intercept | Body contains `add /skus/{id}/regional_prices/-`, `replace /skus/{id}/regional_prices/{id}/original_price`, `remove /skus/{id}/regional_prices/{id}` |
| 67 | should not generate any ops when no changes were made | Load product, do not modify | Click Save (if enabled) | No PATCH sent (or empty patch). Save button may be disabled when form is clean |

### Edit-Specific Variation and SKU Scenarios

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 68 | should add a new property value and create new SKUs in edit mode | Product has Color (Black, White) × DPI (800) | Add "Red" to Color | New value "Red" appears as tag. SKU matrix gains new row for Red/800. New SKU has empty price/stock to fill |
| 69 | should remove a property value and show affected SKU warning | Product has Color (Black, White, Red) with 6 SKUs | Click remove on "Red" tag | Warning: "Removing 'Red' will delete 2 SKUs." Confirm removes Red + associated SKU rows |
| 70 | should edit SKU price inline and track as dirty change | Product loaded with SKU at $24.99 | Click price cell, change to $29.99 | Cell updates. Dirty indicator shows 1 change. Save generates `replace /skus/{id}/price` op |
| 71 | should toggle SKU active state and track change | SKU is active | Click is_active toggle | Toggle switches to inactive. Dirty state. Save generates `replace /skus/{id}/is_active` op |
| 72 | should edit SKU code and detect duplicate on save | Two SKUs exist | Set both to same sku_code "DUP-01" | Server returns 422: "The sku_code 'DUP-01' is already used by another SKU." Error on second SKU row |

### Edit-Specific Gallery Scenarios

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 73 | should add new gallery images in edit mode and generate add ops | Product has 2 images | Add 2 more images via asset picker | 4 images total. Save generates `add /gallery/-` ops for 2 new items |
| 74 | should remove gallery image in edit mode and generate remove op | Product has 3 images | Remove second image | 2 images remain. Save generates `remove /gallery/{id}` op |
| 75 | should change primary image in edit mode and generate replace ops | First image is primary | Set second image as primary | Save generates two ops: `replace /gallery/{oldId}/is_primary false` and `replace /gallery/{newId}/is_primary true` |
| 76 | should reorder gallery in edit mode and generate position ops | 4 images in order A, B, C, D | Drag D to position 0 | Order becomes D, A, B, C. Save generates position replace ops for all 4 items |

### Edit-Specific Regional Pricing Scenarios

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 77 | should add regional pricing to existing SKU in edit mode | Product has SKU without regional pricing | Add AE pricing (original=149, stock=50) | Regional row added. Save generates `add /skus/{id}/regional_prices/-` op |
| 78 | should edit existing regional price in edit mode | SKU has AE pricing at 149.00 | Change to 199.00 | Save generates `replace /skus/{id}/regional_prices/{id}/original_price` with value 199.00 |
| 79 | should remove regional pricing from SKU in edit mode | SKU has AE and US pricing | Remove US pricing | Save generates `remove /skus/{id}/regional_prices/{id}` op |
| 80 | should set discount price on existing regional entry | Regional price has only original_price | Enter discount_price 129.00 | Save generates `replace /skus/{id}/regional_prices/{id}/discount_price` with 129.00 |
| 81 | should clear discount price (set to null) | Regional price has discount_price | Clear discount field | Save generates `replace .../discount_price` with null |

### Edit-Specific Specification Scenarios

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 82 | should edit specification value in edit mode | Product has Brand="Logitech" | Change to "Razer" | Save generates `replace /specifications/{id}/value` with "Razer" |
| 83 | should add new specification in edit mode | Product has 3 specs | Click "+ Add Custom Specification", fill name and value | Save generates `add /specifications/-` op |
| 84 | should remove specification in edit mode | Product has 5 specs | Remove "Weight" spec | Save generates `remove /specifications/{id}` op |
| 85 | should reorder specifications in edit mode | Specs: Brand, Connectivity, Weight | Drag Weight to position 1 | Save generates position replace ops for affected specs |

### PATCH Document Validation Errors

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 86 | should show error when patch document is not a JSON array | Intercept to return 422 "Patch document must be a JSON array" | Save | Toast: "Invalid patch format." This is a programming error indicator |
| 87 | should show error when operation is missing op field | Intercept to return 422 "Operation at index 0 missing required field: op" | Save | Toast with operation index error |
| 88 | should show error when op is unsupported | Intercept to return 422 "unsupported op 'move'" | Save | Toast: "Unsupported operation at index {i}" |
| 89 | should show error when path is not addressable | Intercept to return 422 "Path '/invalid' is not a valid target" | Save | Toast with invalid path error |
| 90 | should show error when replace target ID does not exist | Replace a spec that was deleted server-side | Save | Error: "Cannot replace '/specifications/{id}' — ID not found." Toast advises reload |
| 91 | should show error when remove target ID does not exist | Remove a gallery item that was deleted server-side | Save | Error: "Cannot remove '/gallery/{id}' — ID not found." Toast advises reload |
| 92 | should show error for value type mismatch | Intercept to return 422 "expected number" for price field | Save | Error mapped to field: "Invalid value at '/skus/{id}/price': expected number" |

### PATCH Post-Operation Limit Errors

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 93 | should show error when property values exceed 50 per property after add | Property "Color" has 49 values, add 2 more | Save | Error: "Property 'Color' cannot have more than 50 values. Current: 51 after changes" |
| 94 | should show error when specifications exceed 50 after add | Product has 49 specs, add 2 more | Save | Error: "Product cannot have more than 50 specifications. Current: 51 after changes" |
| 95 | should show error when add SKU references invalid property_value_ids | Add SKU referencing non-existent property value | Save | Error: "The SKU references property values that do not exist: {ids}" |
| 96 | should show error when _tempId reference is not found | SKU uses property_value_refs with unknown _tempId | Save | Error: "The temporary reference '{ref}' was not found in any add operation" |

### PATCH Gallery-Specific Edit Errors

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 97 | should show error when setting is_primary on video via replace | Intercept 422 for is_primary on video | Save | Error: "Only images can be marked as primary" |
| 98 | should show error when changing type to video on primary item | Change primary image type to video | Save | Error: "Cannot change type to video on primary gallery item. Reassign primary first" |
| 99 | should show error when changing type to video without thumbnail | Change image to video type without setting thumbnail | Save | Error: "Cannot change type to video without a thumbnail_url. Set thumbnail_url first" |
| 100 | should show error when adding video without thumbnail in edit mode | Add gallery video without selecting thumbnail | Save | Error: "Gallery video must include a thumbnail_url" |
| 101 | should show error when adding video with is_primary true | Add video gallery item with is_primary=true | Save | Error: "Only images can be marked as primary" |

### PATCH Regional Pricing Edit Errors

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 102 | should show error when adding regional price with invalid region_id | Add regional pricing with non-existent region | Save | Error: "Region ID {id} does not exist" |
| 103 | should show error when adding duplicate region to same SKU | SKU already has AE pricing, add AE again | Save | Error: "Region ID 3 is already assigned to this SKU" |
| 104 | should show error when regional price missing required fields | Add regional price without original_price | Save | Error: "Regional price must include region_id, original_price, and stock" |
| 105 | should show error when regional original_price below minimum | Set regional original_price to 0.00 | Save | Error: "Regional original price must be at least 0.01" |

### Category Change in Edit Mode

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 106 | should show category change warning in edit mode | Product has category with properties and SKUs | Click clear on category | Warning: "Changing the category will reset all variation properties, remove all SKUs, clear category-specific specs. Basic info and gallery preserved." |
| 107 | should reset properties and SKUs when category change is confirmed in edit | Product has properties, SKUs, specs | Confirm category change, select new category | Properties cleared. SKU matrix cleared. Specs reset to new schema. Title and gallery preserved |
| 108 | should show schema validation error after category change on save | Change category, existing specs don't match new schema | Save | Error: "Property/specification '{name}' does not conform to the new category schema" |
