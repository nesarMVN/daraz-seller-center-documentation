# Product Create Flow — E2E Test Architecture

## Page Objects

### ProductCreatePage

```
File: e2e/pages/product-create.page.ts
Class: ProductCreatePage
```

**Constructor parameter:** `page: Page`

**Locators:**

| Locator | Selector | Description |
|---------|----------|-------------|
| `heading` | `page.getByRole('heading', { name: /create product/i })` | Page heading |
| `breadcrumb` | `page.getByText('Products > Create Product')` | Breadcrumb |
| `categorySection` | `page.getByRole('region', { name: /category/i })` | Category section |
| `basicInfoSection` | `page.getByRole('region', { name: /basic info/i })` | Basic information section |
| `variationsSection` | `page.getByRole('region', { name: /variations/i })` | Variations & SKUs section |
| `specificationsSection` | `page.getByRole('region', { name: /specifications/i })` | Specifications section |
| `gallerySection` | `page.getByRole('region', { name: /gallery/i })` | Gallery section |
| `regionalPricingSection` | `page.getByRole('region', { name: /regional pricing/i })` | Regional pricing section |
| `actionBar` | `page.getByTestId('action-bar')` | Sticky bottom action bar |
| `saveDraftButton` | `actionBar.getByRole('button', { name: /save as draft/i })` | Save as draft button |
| `submitButton` | `actionBar.getByRole('button', { name: /submit for review/i })` | Submit for review button |
| `titleInput` | `page.getByLabel(/product title/i)` | Title input field |
| `titleCharCount` | `page.getByTestId('title-char-count')` | Character counter for title |
| `descriptionEditor` | `page.getByTestId('description-editor')` | Rich text editor |
| `completionSidebar` | `page.getByTestId('completion-sidebar')` | Form completion sidebar |
| `progressBar` | `completionSidebar.getByRole('progressbar')` | Overall progress bar |
| `qualityScore` | `completionSidebar.getByTestId('quality-score')` | Quality score text |
| `autoSaveIndicator` | `page.getByTestId('auto-save-indicator')` | Auto-save status text |
| `recoveryPrompt` | `page.getByTestId('draft-recovery-prompt')` | Draft recovery prompt |
| `sourceProductBadge` | `page.getByTestId('source-product-badge')` | Source product info badge |
| `previewModal` | `page.getByRole('dialog', { name: /review product/i })` | Pre-submit preview modal |
| `previewConfirmButton` | `previewModal.getByRole('button', { name: /confirm.*submit/i })` | Confirm button in preview |
| `previewBackButton` | `previewModal.getByRole('button', { name: /back to edit/i })` | Back button in preview |
| `toast` | `page.getByRole('status')` | Toast notification |
| `errorSummary` | `page.getByTestId('error-summary-bar')` | Error summary sticky bar |
| `loadingState` | `page.getByTestId('source-product-loading')` | Source product loading skeleton |

**Actions:**

| Method | Description |
|--------|-------------|
| `async goto()` | Navigates to `/products/create` |
| `async gotoWithSource(sourceId: string)` | Navigates to `/products/create?source_product_id={id}` |
| `async fillTitle(title: string)` | Types in title input |
| `async clearTitle()` | Clears title input |
| `async fillDescription(html: string)` | Types in description editor |
| `async clickSaveDraft()` | Clicks save as draft button |
| `async clickSubmitForReview()` | Clicks submit for review button |
| `async confirmSubmit()` | Confirms in preview modal |
| `async cancelSubmit()` | Goes back from preview modal |
| `async waitForSaveDraftResponse()` | Waits for POST response after save |
| `async waitForSubmitResponse()` | Waits for POST response after submit |
| `async clickSidebarSection(name: string)` | Clicks section name in sidebar to scroll |
| `async restoreDraft()` | Clicks "Restore Draft" in recovery prompt |
| `async discardDraft()` | Clicks "Discard" in recovery prompt |

### SkuBuilderComponent

```
File: e2e/pages/sku-builder.component.ts
Class: SkuBuilderComponent
```

**Locators:**

| Locator | Selector | Description |
|---------|----------|-------------|
| `addPropertyButton` | `section.getByRole('button', { name: /add property/i })` | Add property button |
| `propertyRows` | `section.getByTestId('property-row')` | Property row containers |
| `skuTable` | `section.getByRole('table')` or `section.getByTestId('sku-matrix-table')` | SKU matrix table |
| `skuRows` | `skuTable.getByRole('row')` | SKU table rows |
| `quickFillButton` | `section.getByRole('button', { name: /quick fill/i })` | Batch fill button |
| `skuCountWarning` | `section.getByText(/sku.*warning|exceed/i)` | SKU explosion warning |
| `emptyState` | `section.getByText(/no.*properties|add.*variation/i)` | Empty variation state |
| `noPropertiesSkuRow` | `section.getByTestId('single-sku-row')` | Single SKU row when no properties |

**Dynamic locators:**

| Method | Returns | Description |
|--------|---------|-------------|
| `propertyNameInput(index: number)` | `Locator` | Property name input at index |
| `propertyValueTags(index: number)` | `Locator` | Value tags for property at index |
| `addValueInput(index: number)` | `Locator` | Input for adding new value to property |
| `removePropertyButton(index: number)` | `Locator` | Remove button for property |
| `skuPriceInput(rowIndex: number)` | `Locator` | Price input in SKU row |
| `skuStockInput(rowIndex: number)` | `Locator` | Stock input in SKU row |
| `skuCodeInput(rowIndex: number)` | `Locator` | SKU code input in row |
| `skuActiveToggle(rowIndex: number)` | `Locator` | Active toggle in SKU row |
| `removeValueButton(propIndex: number, valueIndex: number)` | `Locator` | Remove button on value tag |

**Actions:**

| Method | Description |
|--------|-------------|
| `async addProperty(name: string, values: string[])` | Adds a property with values |
| `async addValue(propertyIndex: number, value: string)` | Adds a value to a property |
| `async removeProperty(index: number)` | Removes a property (confirms dialog) |
| `async removeValue(propIndex: number, valIndex: number)` | Removes a value |
| `async setSkuPrice(row: number, price: string)` | Sets price for a SKU row |
| `async setSkuStock(row: number, stock: string)` | Sets stock for a SKU row |
| `async setSkuCode(row: number, code: string)` | Sets SKU code |
| `async toggleSkuActive(row: number)` | Toggles active state |
| `async quickFill(field: string, value: string)` | Batch fills price or stock |
| `async dragProperty(from: number, to: number)` | Reorders property via drag |
| `async dragValue(propIndex: number, from: number, to: number)` | Reorders value via drag |
| `async dragSkuRow(from: number, to: number)` | Reorders SKU row via drag |
| `getSkuCount(): Locator` | Returns SKU count display |

### RegionalPricingComponent

```
File: e2e/pages/regional-pricing.component.ts
Class: RegionalPricingComponent
```

**Locators:**

| Locator | Selector | Description |
|---------|----------|-------------|
| `section` | `page.getByRole('region', { name: /regional pricing/i })` | Section container |
| `skuAccordions` | `section.getByTestId('sku-accordion')` | Per-SKU expandable sections |
| `addRegionButton` | `Locator` | Add region pricing button per SKU |
| `bulkCopyButton` | `section.getByRole('button', { name: /copy.*pricing/i })` | Bulk copy across SKUs |

**Dynamic locators:**

| Method | Returns | Description |
|--------|---------|-------------|
| `skuSection(index: number)` | `Locator` | Expandable section for SKU at index |
| `regionRow(skuIndex: number, regionIndex: number)` | `Locator` | Region pricing row |
| `originalPriceInput(skuIndex: number, regionIndex: number)` | `Locator` | Original price input |
| `discountPriceInput(skuIndex: number, regionIndex: number)` | `Locator` | Discount price input |
| `regionStockInput(skuIndex: number, regionIndex: number)` | `Locator` | Region stock input |
| `regionActiveToggle(skuIndex: number, regionIndex: number)` | `Locator` | Region active toggle |
| `removeRegionButton(skuIndex: number, regionIndex: number)` | `Locator` | Remove region pricing |

**Actions:**

| Method | Description |
|--------|-------------|
| `async expandSku(index: number)` | Expands a SKU accordion |
| `async addRegionPrice(skuIndex: number, regionName: string, price: string, stock: string)` | Adds regional pricing to a SKU |
| `async removeRegionPrice(skuIndex: number, regionIndex: number)` | Removes regional pricing |
| `async setOriginalPrice(skuIndex: number, regionIndex: number, price: string)` | Sets original price |
| `async setDiscountPrice(skuIndex: number, regionIndex: number, price: string)` | Sets discount price |
| `async setRegionStock(skuIndex: number, regionIndex: number, stock: string)` | Sets region stock |
| `async bulkCopyPricing(sourceSkuIndex: number)` | Copies pricing from one SKU to all |

---

## Helpers

- `productTestData.fullProduct()` — Complete payload for API-created reference products
- `productTestData.properties()` — Property data for form-filling
- `productTestData.categoryId()` — Known category with schema
- `PRODUCT_CONSTANTS.CATEGORY_WITH_SCHEMA` — Category metadata for assertions
- `PRODUCT_CONSTANTS.LIMITS` — Validation boundaries

---

## Fixture Wiring

```typescript
productCreatePage: async ({ page }, use) => {
  await use(new ProductCreatePage(page))
}

skuBuilderComponent: async ({ page }, use) => {
  await use(new SkuBuilderComponent(page))
}

regionalPricingComponent: async ({ page }, use) => {
  await use(new RegionalPricingComponent(page))
}
```

---

## Spec Files

| Spec File | Concern | Estimated Scenarios |
|-----------|---------|-------------------|
| `product-create-page.spec.ts` | Page layout, section presence | 6 |
| `product-create-category.spec.ts` | Category picker in create context | (covered in 04-category-picker.md) |
| `product-create-basic-info.spec.ts` | Title and description fields | 12 |
| `product-create-variations.spec.ts` | Property and SKU builder | 16 |
| `product-create-specs.spec.ts` | Specification form | 8 |
| `product-create-gallery.spec.ts` | Gallery manager in create | (covered in 05-gallery-manager.md) |
| `product-create-regional.spec.ts` | Regional pricing | 8 |
| `product-create-draft.spec.ts` | Save as draft workflow | 6 |
| `product-create-submit.spec.ts` | Submit for review workflow | 6 |
| `product-create-validation.spec.ts` | All validation error scenarios | 20 |
| `product-create-source.spec.ts` | Source product pre-fill | 7 |
| `product-create-autosave.spec.ts` | Auto-save and recovery | 5 |

---

## Test Scenarios

### Page Layout (`product-create-page.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 1 | should display create product page with heading and breadcrumb | Navigate to `/products/create` | Observe page | Heading "Create Product" visible. Breadcrumb "Products > Create Product" visible |
| 2 | should display all 6 form sections | Navigate to create page | Observe sections | Sections visible: Category, Basic Information, Variations & SKUs, Specifications, Gallery, Regional Pricing |
| 3 | should display sticky action bar with save and submit buttons | Navigate to create page | Observe bottom bar | "Save as Draft" and "Submit for Review" buttons visible in sticky action bar |
| 4 | should display form completion sidebar with progress tracker | Navigate to create page | Observe sidebar | Sidebar shows section names with status indicators (empty/partial/complete), progress bar, quality score |
| 5 | should disable all sections except category before category selection | Navigate to create page (no category) | Observe sections | Basic Info shows "Select a category first". Variations, Specs, Gallery, Regional sections disabled or collapsed |
| 6 | should scroll to section when sidebar section name is clicked | Category selected, all sections visible | Click "Gallery" in sidebar | Page scrolls to gallery section. Gallery section in viewport |

### Basic Information (`product-create-basic-info.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 7 | should display title input with label and asterisk (required) | Select category | Observe basic info | "Product Title *" label visible. Input placeholder: "Enter product title..." |
| 8 | should show character counter on title focus | Focus title input | Observe counter | Counter appears: "0/255 characters" |
| 9 | should update character counter as user types | Focus and type "Hello" | Observe counter | Counter shows "5/255 characters" |
| 10 | should show orange counter at 230 characters (approaching limit) | Type 230 characters | Observe counter | Counter text turns orange |
| 11 | should show red bold counter at 256+ characters (over limit) | Type 260 characters | Observe counter | Counter shows "260/255 characters" in red bold |
| 12 | should show green border and check icon for valid title | Type a valid title (1-255 chars) | Observe input | Green border on input. Check icon on right |
| 13 | should show description rich text editor with toolbar | Select category | Observe editor | Rich text editor visible with toolbar: Bold, Italic, Underline, H1, H2, Ul, Ol, Link, Img |
| 14 | should accept HTML formatting in description | Type and format text with bold and lists | Observe editor | Formatted text visible. Toolbar buttons show active state for applied formatting |
| 15 | should strip unsafe HTML on paste | Paste HTML with script tags | Observe editor | Script tags stripped. Safe formatting preserved (bold, italic, lists, links) |
| 16 | should truncate pasted content exceeding 65535 characters | Paste very long content | Observe | Content trimmed. Toast: "Content was trimmed to fit the 65,535 char limit" |
| 17 | should show inline help tooltip on title label hover | Hover info icon next to title | Observe tooltip | Tooltip shows: "Use a clear, descriptive title. Include key features, brand name, and product type" |
| 18 | should show disabled state for title before category selection | Navigate to create (no category) | Observe title | Input disabled, gray background, hint: "Select a category first" |

### Variations & SKUs (`product-create-variations.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 19 | should show empty state when no properties are defined | Select category | Observe variations section | Empty state message. "Add Property" button visible |
| 20 | should add a property with schema-defined name from dropdown | Category with schema selected | Click "Add Property", select "Color" from dropdown | Property row added with name "Color". Value input appears |
| 21 | should add values to a property as tags | Property "Color" exists | Type "Black" and press Enter | "Black" tag appears. Can add more values: "White", "Red" |
| 22 | should generate SKU matrix from cartesian product of property values | Add Color (Black, White) and DPI (800, 1600) | Observe SKU table | 4 SKU rows generated: Black/800, Black/1600, White/800, White/1600 |
| 23 | should allow inline editing of SKU price, stock, code, and active toggle | SKU matrix visible | Click price cell, type "24.99" | Price updated. Can also edit stock, sku_code, toggle is_active |
| 24 | should show price range hint below price input | Focus on a SKU price cell | Observe hint | Hint shows "$0.01 – $9,999,999,999.99" |
| 25 | should allow maximum 3 properties | Add 3 properties | Observe "Add Property" button | Button disabled or hidden after 3 properties |
| 26 | should allow maximum 50 values per property | Add property with 50 values | Try to add 51st value | Input disabled or error: max 50 values per property |
| 27 | should show SKU explosion warning when combinations exceed 100 | Add 3 properties with many values approaching 100+ combos | Observe | Warning: "This will generate X SKUs (max 100)" before generating |
| 28 | should show confirmation dialog when removing a property | Properties and SKUs exist | Click remove on a property | Dialog: "Removing this property will delete X SKUs that use it. Are you sure?" |
| 29 | should cascade-remove related SKUs when property is removed and confirmed | 2 properties, 6 SKUs | Remove one property, confirm | Property removed. SKU matrix regenerated with remaining property only. SKU count decreased |
| 30 | should reorder property values via drag and drop | Property with values Black, White, Red | Drag Red to position 0 | Order: Red, Black, White. SKU matrix regenerated in new order |
| 31 | should quick-fill price for all SKUs | Multiple SKUs exist | Click "Quick Fill", set price to 29.99 | All SKU price cells updated to 29.99 |
| 32 | should quick-fill stock for all SKUs | Multiple SKUs | Quick fill stock = 100 | All stock cells updated to 100 |
| 33 | should show single SKU row when no properties are defined | Select category, no properties added | Observe | Single SKU row visible with price, stock, sku_code inputs. No property columns |
| 34 | should allow free-text property names when category has no schema | Select category without schema | Click "Add Property" | Free-text name input instead of dropdown. No schema constraint on values |

### Specifications (`product-create-specs.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 35 | should display schema-driven specification fields after category selection | Select category with schema (Mouse) | Observe specs section | Fields appear: Brand (text, required), Connectivity (select, required), Sensor Type (select, optional), etc. Required fields marked with asterisk |
| 36 | should show dropdown with schema options for select-type specs | Category with schema selected | Click Connectivity dropdown | Options: Wired, Wireless (2.4GHz), Bluetooth, Dual Mode |
| 37 | should allow free-text input for text-type specs | Category with schema | Focus on Brand input | Free text input. Type "Logitech" |
| 38 | should allow adding custom specifications beyond schema | Schema specs visible | Click "+ Add Custom Specification" | New row with empty Name and Value inputs. Type=text by default |
| 39 | should show free-form specification fields when category has no schema | Select category without schema | Observe specs section | No pre-populated fields. "+ Add Specification" button. Each spec has Name + Value free-text inputs |
| 40 | should reorder specifications via drag and drop | Multiple specs exist | Drag "Weight" from position 3 to position 1 | Spec order updated. Position values recalculated |
| 41 | should remove a custom specification | Custom spec added | Click remove (x) on spec row | Spec removed from list |
| 42 | should enforce max 50 specifications | Add 50 specs | Try to add 51st | Add button disabled or error |

### Regional Pricing (`product-create-regional.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 43 | should display per-SKU expandable sections for regional pricing | SKUs exist | Observe regional pricing section | Each SKU has an expandable accordion. SKU label shows property values |
| 44 | should add regional pricing for a specific region to a SKU | Expand SKU accordion | Click "Add Region", select "UAE (AE)", enter price 149.00, stock 50 | Regional pricing row added: AE region, original_price=149.00, stock=50 |
| 45 | should show all 6 available regions in add dropdown | Expand SKU, click "Add Region" | Observe dropdown | BD, IN, AE, US, EU, AU options visible |
| 46 | should allow setting discount price (optional) for a regional entry | Regional price row exists | Enter discount_price 129.00 | Discount price saved. Must be <= original_price |
| 47 | should remove regional pricing for a SKU | Regional price row exists | Click remove on regional row | Regional pricing removed for that region |
| 48 | should prevent duplicate region_id per SKU | SKU has AE pricing | Try to add AE again | AE disabled in dropdown or error: "Region already assigned" |
| 49 | should copy regional pricing from one SKU to all SKUs | First SKU has BD and AE pricing | Click "Copy Pricing" | All other SKUs receive same BD and AE pricing structure |
| 50 | should show currency formatting based on region | Add pricing for US and AE regions | Observe price display | US shows USD format, AE shows AED format via Intl.NumberFormat |

### Save as Draft (`product-create-draft.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 51 | should save product as draft with minimal data (category + title) | Fill category and title only | Click "Save as Draft" | API: POST /seller-products with status=draft. Response 201. Toast: "Product saved as draft". Navigate to edit page |
| 52 | should save product as draft with full data | Fill all sections (category, title, desc, properties, SKUs, specs, gallery, regional) | Click "Save as Draft" | API: POST with all data, status=draft. 201 response. Navigate to edit page |
| 53 | should disable save button when category is not selected | No category selected | Observe save button | "Save as Draft" button disabled. Tooltip: "Select a category and enter a title to save" |
| 54 | should disable save button when title is empty | Category selected, title empty | Observe save button | "Save as Draft" disabled |
| 55 | should show loading state on save button during API call | Click "Save as Draft" | Observe button during request | Button shows spinner and "Saving..." text. Both buttons disabled |
| 56 | should show success flash on button after save completes | Draft saved successfully | Observe button | Brief green flash with check icon, then navigate to edit page |

### Submit for Review (`product-create-submit.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 57 | should show preview modal before submitting when all required fields filled | Fill all required: category, title, 1+ SKU, primary image, required specs | Click "Submit for Review" | Preview modal opens showing: category, title, description preview, variation/SKU summary, spec count, gallery count, regional pricing summary |
| 58 | should submit product when confirmed in preview modal | Preview modal open | Click "Confirm & Submit" | API: POST with status=pending. 201 response. Toast: "Product submitted for review". Navigate to product list |
| 59 | should return to form when "Back to Edit" is clicked in preview | Preview modal open | Click "Back to Edit" | Modal closes. Form still editable. No API call made |
| 60 | should disable submit button when required fields are missing | Category and title filled but no SKU | Observe submit button | "Submit for Review" disabled. Tooltip lists missing fields |
| 61 | should show loading state on submit button during API call | Click submit, confirm | Observe button | Button: "Submitting..." with spinner. All buttons disabled |
| 62 | should invalidate product list and status counts after successful submit | Submit completes | Navigate to product list | Product appears in Pending tab. Status counts updated |

### Validation Errors (`product-create-validation.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 63 | should show inline error when title is empty on submit | Fill everything except title | Submit | Red border on title. Error: "The title field is required." Scroll to title field |
| 64 | should show inline error when title exceeds 255 characters | Fill title with 260 chars | Submit | Error: "The title must not exceed 255 characters." Red counter |
| 65 | should show error when description exceeds 65535 characters | Fill very long description | Submit | Error: "The description must not exceed 65535 characters" |
| 66 | should show error when category is not a leaf node | Server returns 422 for non-leaf | Intercept response | Error: "Category must be a leaf category (no children)" |
| 67 | should show error when shipping category is not found (404) | Category deleted between selection and save | Submit | Toast: "Category not found. Please re-select category" |
| 68 | should show error when required property is missing on submit as pending | Category schema requires "Color", not added | Submit for review | Error: "The required property 'Color' is missing" |
| 69 | should show error when property name is not in category schema | Add property "Weight" (not in Mouse schema) | Submit | Error: "The property 'Weight' is not defined in the category schema" |
| 70 | should show error when property value is not in schema options | Add schema property with invalid value | Submit | Error: "The value '{value}' is not a valid option for '{name}'" |
| 71 | should show error for duplicate property names | Add two properties with same name | Submit | Error: "Duplicate property name: '{name}'" |
| 72 | should show error for duplicate values within a property | Add "Black" twice to Color property | Submit | Error: "Duplicate value 'Black' in property 'Color'" |
| 73 | should show error when properties exceed 3 | Try to add 4th property via client manipulation | Submit | Error: "The properties must not have more than 3 items" |
| 74 | should show error when SKU price is below 0.01 | Set SKU price to 0.00 | Submit | Error: "The skus.{i}.price must be at least 0.01." Red border on price cell |
| 75 | should show error when SKU price exceeds max | Set SKU price above 9999999999.99 | Submit | Error: "must not be greater than 9999999999.99" |
| 76 | should show error for duplicate SKU combinations | Two SKUs with same property_values | Submit | Error: "Duplicate SKU: two SKUs reference the same combination of property values" |
| 77 | should show error for duplicate sku_code | Set same sku_code on two SKUs | Submit | Error: "The sku_code '{code}' is already used by another SKU" |
| 78 | should show error when required specification is missing on pending submit | Required spec "Brand" not filled | Submit for review | Error: "The required specification 'Brand' is missing" |
| 79 | should show error when select spec value is not in options | Select spec with invalid value (server-side) | Submit | Error: "The specification '{name}' requires one of: {options}" |
| 80 | should show error for duplicate specification names | Add two specs with same name | Submit | Error: "Duplicate specification name: '{name}'" |
| 81 | should show error when gallery exceeds 20 items | Add 21 gallery items | Submit | Error: "The gallery must not have more than 20 items" |
| 82 | should show error when no primary image is set | Gallery has images but none marked primary | Submit for review | Error: "Exactly one gallery image must be marked as primary" |
| 83 | should show error when video is marked as primary | Video item with is_primary=true | Submit | Error: "Only images can be marked as primary" |
| 84 | should show error when video has no thumbnail | Video without thumbnail_url | Submit | Error: "Gallery video must include a thumbnail_url" |
| 85 | should show error when gallery asset is not found | Asset deleted between selection and submit | Submit | Error: "The asset '{asset_id}' was not found" |
| 86 | should show error when regional price region_id does not exist | Invalid region_id | Submit | Error: "Region ID {id} does not exist" |
| 87 | should show error for duplicate region_id within a SKU | Same region added twice to a SKU | Submit | Error: "Region ID {id} is already assigned to this SKU" |
| 88 | should show error when SKUs exceed 100 | Create >100 SKU combinations | Submit | Error: "The skus must not have more than 100 items" |
| 89 | should show error when specifications exceed 50 | Add 51 specifications | Submit | Error: "The specifications must not have more than 50 items" |
| 90 | should scroll to first error section on validation failure | Multiple errors exist | Submit | Page scrolls to first section with error. Error summary bar visible: "X validation errors" with scroll-to-first link |
| 91 | should map server 422 error paths to correct form fields | Server returns errors with dot-notation paths | Intercept 422 response | Each error mapped to correct field: `properties.0.name` -> property 0 name input, `skus.2.price` -> SKU row 2 price cell |
| 92 | should show toast for 400 transaction failure | Server returns 400 | Submit | Toast: "Failed to create product. Please try again" |

### Source Product Pre-fill (`product-create-source.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 93 | should show loading state when source_product_id is in URL | Navigate to `/products/create?source_product_id=01KJJJD0M7E89...` | Observe loading | Skeleton loader visible: "Loading source product..." |
| 94 | should pre-fill form fields from source product data | Source product fetched successfully | Observe form | Title, description, property names/values, SKU prices, specifications pre-filled. Category NOT pre-filled (still needs manual selection) |
| 95 | should show source product badge with product name | Source loaded | Observe badge | Badge: "Sourced from: {title} ({ulid})" with note about adjusting prices |
| 96 | should handle source product not found (404) gracefully | source_product_id points to deleted product | Observe | Toast: "Source product not found. You can still create your product manually." Form reverts to empty. source_product_id stripped from URL |
| 97 | should show retry option on network error when fetching source | Network error during source fetch | Observe | Toast with "Retry" and "Skip" buttons. Retry re-fetches. Skip falls back to empty form |
| 98 | should redirect to clean URL for invalid source_product_id format | Navigate with malformed ULID | Observe | Redirected to `/products/create` (no source param). Toast: "Invalid source product reference" |
| 99 | should still require category selection even with source pre-fill | Source pre-fills title and SKUs | Observe category | Category field empty. "Select a category" still required. Sections below still locked until category chosen |

### Auto-Save and Recovery (`product-create-autosave.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 100 | should auto-save form state to localStorage after 60 seconds of inactivity | Fill some form fields | Wait 60+ seconds | localStorage contains `product-draft-{category_id}` with form snapshot |
| 101 | should show auto-save indicator in action bar | Auto-save triggers | Observe action bar | Text: "Auto-saving draft..." then "Draft auto-saved at {time}" |
| 102 | should show recovery prompt when returning to create with existing localStorage draft | Save draft to localStorage, navigate away, return | Observe page | Prompt: "Unsaved draft found from X minutes ago. [Restore Draft] [Discard]" |
| 103 | should restore form data when "Restore Draft" is clicked | Recovery prompt visible | Click "Restore Draft" | All previously entered form data restored. Prompt dismissed |
| 104 | should clear localStorage draft when "Discard" is clicked | Recovery prompt visible | Click "Discard" | Empty form shown. localStorage draft removed |

### Form Completion Sidebar

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 105 | should show 0% progress when no sections are filled | Navigate to create (no data) | Observe sidebar | Progress bar at 0%. All sections show empty (circle) icon |
| 106 | should update section status as fields are filled | Fill category and title | Observe sidebar | Category shows complete (filled circle + check). Basic Info shows partial (half circle) if only title filled |
| 107 | should show quality score tier based on data completeness | Fill required fields only vs fill everything | Observe quality | Required only: "Poor" or "Good". All optional filled: "Excellent" with 75%+ score |

### Button States

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 108 | should disable both save and submit buttons during mutation | Click save | Observe both buttons | Both "Save as Draft" and "Submit for Review" disabled during API call |
| 109 | should show correct tooltip on disabled submit button | Required fields incomplete | Hover submit button | Tooltip: "Complete all required fields: category, title, 1+ SKU, primary image, required specs" |
