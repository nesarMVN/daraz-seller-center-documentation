# Product List Page — E2E Test Architecture

## Page Object

### ProductListPage

```
File: e2e/pages/product-list.page.ts
Class: ProductListPage
```

**Constructor parameter:** `page: Page`

**Locators:**

| Locator | Selector | Description |
|---------|----------|-------------|
| `heading` | `page.getByRole('heading', { name: /manage products/i })` | Page heading |
| `breadcrumb` | `page.getByText('Products > Manage Products')` | Breadcrumb navigation |
| `newProductButton` | `page.getByRole('button', { name: /new product/i })` | Create product navigation button |
| `searchInput` | `page.getByRole('searchbox', { name: /search products/i })` | Search input field |
| `searchClearButton` | `page.getByRole('button', { name: /clear search/i })` | Clear search (x) button |
| `categoryDropdown` | `page.getByRole('combobox', { name: /category/i })` | Category filter dropdown |
| `regionDropdown` | `page.getByRole('combobox', { name: /region/i })` | Region filter dropdown |
| `sortFieldDropdown` | `page.getByRole('combobox', { name: /sort by/i })` | Sort field dropdown |
| `sortOrderDropdown` | `page.getByRole('combobox', { name: /sort order/i })` | Sort order dropdown |
| `tabList` | `page.getByRole('tablist')` | Status tabs container |
| `productTable` | `page.getByRole('table')` | Product data table |
| `tableHeaders` | `productTable.getByRole('columnheader')` | Table column headers |
| `tableRows` | `productTable.getByRole('row')` | All table rows (including header) |
| `headerCheckbox` | `page.getByRole('checkbox', { name: /select all/i })` | Select-all checkbox in header |
| `paginationNav` | `page.getByRole('navigation', { name: /pagination/i })` | Pagination navigation |
| `prevButton` | `paginationNav.getByRole('button', { name: /prev/i })` | Previous page button |
| `nextButton` | `paginationNav.getByRole('button', { name: /next/i })` | Next page button |
| `pageInfo` | `page.getByText(/page \d+ of \d+/i)` | "Page X of Y" text |
| `perPageDropdown` | `page.getByRole('combobox', { name: /per page/i })` | Per-page selector |
| `bulkActionBar` | `page.getByRole('toolbar', { name: /bulk actions/i })` | Bulk action toolbar |
| `bulkActivateButton` | `bulkActionBar.getByRole('button', { name: /activate/i })` | Bulk activate button |
| `bulkDeactivateButton` | `bulkActionBar.getByRole('button', { name: /deactivate/i })` | Bulk deactivate button |
| `bulkDeleteButton` | `bulkActionBar.getByRole('button', { name: /delete/i })` | Bulk delete button |
| `bulkCancelButton` | `bulkActionBar.getByRole('button', { name: /cancel/i })` | Cancel selection button |
| `selectedCount` | `bulkActionBar.getByText(/\d+ products? selected/i)` | Selected products count |
| `emptyStateContainer` | `page.getByTestId('empty-state')` | Empty state display |
| `emptyStateTitle` | `emptyStateContainer.getByRole('heading')` | Empty state heading |
| `emptyStateAction` | `emptyStateContainer.getByRole('button')` | Empty state action button |
| `loadingSkeletons` | `page.getByTestId('skeleton-row')` | Loading skeleton rows |
| `errorContainer` | `page.getByTestId('error-state')` | Error state display |
| `retryButton` | `errorContainer.getByRole('button', { name: /retry/i })` | Error retry button |
| `confirmDialog` | `page.getByRole('alertdialog')` | Confirmation dialog (delete/deactivate) |
| `confirmDialogConfirm` | `confirmDialog.getByRole('button', { name: /confirm|delete|deactivate|activate/i })` | Dialog confirm button |
| `confirmDialogCancel` | `confirmDialog.getByRole('button', { name: /cancel/i })` | Dialog cancel button |
| `toast` | `page.getByRole('status')` | Toast notification |

**Dynamic locators:**

| Method | Returns | Description |
|--------|---------|-------------|
| `tab(status: string)` | `Locator` | `page.getByRole('tab', { name: new RegExp(status, 'i') })` |
| `tabBadge(status: string)` | `Locator` | Badge count inside a status tab |
| `productRow(index: number)` | `Locator` | Table body row at index |
| `rowCheckbox(index: number)` | `Locator` | Checkbox in product row |
| `rowTitle(index: number)` | `Locator` | Product title in row |
| `rowStatus(index: number)` | `Locator` | Status badge in row |
| `rowPrice(index: number)` | `Locator` | Price text in row |
| `rowSkuCount(index: number)` | `Locator` | SKU count in row |
| `rowContextMenu(index: number)` | `Locator` | Three-dot context menu button |
| `contextMenuItem(action: string)` | `Locator` | Menu item (Edit, Duplicate, Activate, Deactivate, Delete) |
| `pageButton(pageNum: number)` | `Locator` | Specific page number button in pagination |
| `statusBadgeTooltip(index: number)` | `Locator` | Tooltip on status badge hover |

**Actions:**

| Method | Description |
|--------|-------------|
| `async goto()` | Navigates to `/products` |
| `async gotoWithParams(params: URLSearchParams)` | Navigates with query params |
| `async clickTab(status: string)` | Clicks a status tab |
| `async search(query: string)` | Types in search, waits for debounce |
| `async clearSearch()` | Clicks clear search button |
| `async selectCategory(name: string)` | Opens category dropdown, selects a category |
| `async clearCategory()` | Clears selected category filter |
| `async selectRegion(name: string)` | Opens region dropdown, selects a region |
| `async clearRegion()` | Clears region filter |
| `async setSortField(field: string)` | Changes sort field |
| `async setSortOrder(order: string)` | Changes sort order |
| `async goToPage(pageNum: number)` | Clicks page number in pagination |
| `async goToNextPage()` | Clicks next page button |
| `async goToPrevPage()` | Clicks previous page button |
| `async selectProduct(index: number)` | Checks the checkbox on a product row |
| `async deselectProduct(index: number)` | Unchecks the checkbox on a product row |
| `async selectAll()` | Clicks the header select-all checkbox |
| `async deselectAll()` | Clicks cancel selection in bulk bar |
| `async openContextMenu(index: number)` | Clicks three-dot menu on a row |
| `async clickContextAction(action: string)` | Clicks an action in context menu |
| `async confirmAction()` | Confirms in dialog |
| `async cancelAction()` | Cancels in dialog |
| `async waitForTableLoad()` | Waits for skeletons to disappear and table rows to appear |
| `async getProductCount(): Promise<number>` | Returns the visible product row count |

---

## Helpers

- `productApiHelper.createProductBatch(request, count, statusDistribution)` — Creates products for list tests
- `productTestData.title()` — Generates unique titles for search tests
- `PRODUCT_CONSTANTS.SORT_FIELDS`, `PRODUCT_CONSTANTS.STATUSES` — Validation values

---

## Fixture Wiring

```typescript
productListPage: async ({ page }, use) => {
  await use(new ProductListPage(page))
}
```

---

## Spec Files

| Spec File | Concern | Estimated Scenarios |
|-----------|---------|-------------------|
| `product-list-page.spec.ts` | Page rendering, layout, element presence | 10 |
| `product-list-tabs.spec.ts` | Status tab navigation and badge counts | 8 |
| `product-list-search.spec.ts` | Search input, debounce, clear | 6 |
| `product-list-filters.spec.ts` | Category, region, sort filters | 8 |
| `product-list-pagination.spec.ts` | Pagination navigation | 5 |
| `product-list-empty-states.spec.ts` | All 5 empty state variations | 5 |
| `product-list-actions.spec.ts` | Context menu single-product actions | 8 |
| `product-list-bulk.spec.ts` | Bulk selection and operations | 10 |
| `product-list-loading.spec.ts` | Skeletons and background refetch | 3 |
| `product-list-errors.spec.ts` | Error states and error recovery | 6 |

---

## Test Scenarios

### Page Rendering (`product-list-page.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 1 | should display page heading and breadcrumb | Create some products via API | Navigate to `/products` | Heading "Manage Products" visible. Breadcrumb "Products > Manage Products" visible |
| 2 | should display all 7 status tabs | Navigate to list page | Observe tab list | Tabs visible: All, Active, Inactive, Draft, Pending, Violation, Deleted. Each tab has role="tab" |
| 3 | should display toolbar with search, category filter, and new product button | Navigate to list page | Observe toolbar | Search input, category dropdown, region dropdown, sort dropdowns, and "+ New Product" button all visible |
| 4 | should display product table with correct column headers | Navigate with products existing | Observe table | Headers visible: checkbox column, Product, Status, Price, SKUs |
| 5 | should display product row with image, title, category, status badge, price, and SKU count | Create a product with gallery image via API | Navigate to list | Row shows: thumbnail image, product title, category path, colored status badge, price (or price range), SKU count |
| 6 | should display pagination controls below table | Create 25+ products | Navigate to list | Pagination nav visible: Prev, page numbers, Next, "Page 1 of N" text, per-page selector |
| 7 | should navigate to create page when new product button is clicked | Navigate to list | Click "+ New Product" button | URL changes to `/products/create` |
| 8 | should show placeholder image for products without gallery | Create a product without gallery via API | Navigate to list | Row shows placeholder icon (gray box) instead of thumbnail |
| 9 | should show "No price" in muted text for products without SKUs | Create a draft product with no SKUs | Navigate to list, filter to draft tab | Price column shows "No price" in gray text. SKU count shows "0" |
| 10 | should display price range when product has SKUs with different prices | Create a product with SKUs at $12.00 and $18.00 | Navigate to list | Price column shows "$12.00 - $18.00" |

### Status Tab Navigation (`product-list-tabs.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 11 | should show badge counts on each status tab matching status-counts API | Create products with various statuses | Navigate to list | Each tab shows correct count in badge. Counts match GET /status-counts response |
| 12 | should filter products by status when a tab is clicked | Create active and draft products | Click "Active" tab | URL updates to `?status=active`. Only active products shown. Pagination reflects active count |
| 13 | should highlight the selected tab with active style | Navigate to list | Click "Inactive" tab | Inactive tab has `aria-selected="true"`. Other tabs have `aria-selected="false"` |
| 14 | should show all non-deleted products when All tab is clicked | Products exist in various statuses | Click "All" tab | URL has no `status` param. All non-deleted products shown |
| 15 | should reset page to 1 when switching tabs | Navigate to page 2 on All tab | Click "Active" tab | URL: `?status=active&page=1`. Page resets to 1 |
| 16 | should clear search and category filters when switching tabs | Search for "mouse" on All tab | Click "Draft" tab | URL: `?status=draft`. Search input cleared. Category filter cleared |
| 17 | should show deleted products with deleted_at date in Deleted tab | Create then delete a product | Click "Deleted" tab | Deleted products shown with deleted_at date. Rows are view-only. No action buttons in context menu |
| 18 | should not show bulk action buttons on Deleted tab | Navigate to Deleted tab | Select products | No bulk activate/deactivate/delete buttons visible. Only "View" in context menu |

### Search (`product-list-search.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 19 | should filter products by title after 300ms debounce | Create products "E2E Mouse Test" and "E2E Keyboard Test" | Type "mouse" in search | After debounce, URL updates with `&title=mouse`. Only "E2E Mouse Test" product visible |
| 20 | should show clear button when search has text | Type a query | Observe search input | Clear (x) button appears on the right side of the input |
| 21 | should clear search and reload all products when clear button is clicked | Search for "mouse" showing filtered results | Click clear (x) button | Search input cleared. `title` param removed from URL. Full product list shown |
| 22 | should show empty state when search returns no results | Products exist but none match query | Type "xyznonexistent" in search | Empty state: "No products match your search" with "Clear Filters" button |
| 23 | should reset page to 1 when search query changes | On page 2 | Type a search query | URL updates with `page=1`. First page of search results shown |
| 24 | should debounce search and not fire API call on every keystroke | Open network inspector | Type "mouse" character by character | Only one API call fires after 300ms of no typing, not one per character |

### Filters and Sorting (`product-list-filters.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 25 | should filter products by category when a leaf category is selected | Create products in different categories | Select "Mouse" category from dropdown | URL updates with `shipping_category_id=<uuid>`. Only products in that category shown. Category dropdown shows full path with clear (x) |
| 26 | should clear category filter when clear button is clicked | Category filter active | Click clear (x) on category dropdown | `shipping_category_id` removed from URL. Full product list restored |
| 27 | should filter products by region when region is selected | Create products with regional pricing | Select "UAE (AE)" region | URL updates with `region_id=3`. Only products with AE pricing shown. Price column shows AE currency |
| 28 | should show "No products in {region}" empty state when no regional pricing exists | No products have AE pricing | Select "UAE (AE)" region | Empty state: "No products in UAE" with "Clear Filter" button |
| 29 | should sort products by price ascending | Create products with different prices | Set sort to "Price" + "Asc" | URL: `sort_by=price&sort_order=asc`. Products ordered by min price ascending |
| 30 | should sort products by title descending | Products exist | Set sort to "Title" + "Desc" | URL: `sort_by=title&sort_order=desc`. Products ordered by title Z-A |
| 31 | should sort products by created date by default | Navigate to list with no params | Observe product order | Default sort: `created_at desc`. Most recently created products first |
| 32 | should reset page to 1 when sort changes | On page 2 | Change sort field | URL updates with `page=1` and new sort params |

### Pagination (`product-list-pagination.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 33 | should navigate to next page when Next button is clicked | Create 25+ products, on page 1 | Click "Next" button | URL updates to `page=2`. New page of products shown. Page info updates |
| 34 | should navigate to previous page when Prev button is clicked | On page 2 | Click "Prev" button | URL updates to `page=1`. First page shown |
| 35 | should navigate to specific page when page number is clicked | On page 1 of 3 | Click page 3 button | URL updates to `page=3`. Third page of products shown. `aria-current="page"` on page 3 button |
| 36 | should disable Prev button on first page | Navigate to page 1 | Observe Prev button | Prev button is disabled |
| 37 | should disable Next button on last page | Navigate to last page | Observe Next button | Next button is disabled |

### Empty States (`product-list-empty-states.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 38 | should show "no products yet" empty state for new seller | No products exist for this seller | Navigate to list | Empty state: "You have no products yet" with "Create Product" button. Badge counts all show 0 |
| 39 | should show "no results" empty state when search/filter has no matches | Products exist | Search for non-matching query | Empty state: "No products match your search" with "Clear Filters" button |
| 40 | should show status-specific empty state when tab has zero products | No violation products exist | Click "Violation" tab | Empty state: "No products with violation status" with message "All your products are in good standing" |
| 41 | should show "no products in region" empty state | No products with BD pricing | Select "Bangladesh" region | Empty state: "No products in Bangladesh" with "Clear Filter" button |
| 42 | should show "no deleted products" empty state | No deleted products | Click "Deleted" tab | Empty state: "No deleted products" with "Your product catalog is intact" |

### Single Product Actions (`product-list-actions.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 43 | should navigate to edit page when product row is clicked | Products in list | Click product title/row | URL changes to `/products/{id}/edit` |
| 44 | should show context menu with status-appropriate actions on three-dot click | Active product in list | Click three-dot menu | Menu shows: Edit, Duplicate, Deactivate, Delete. No "Activate" (already active) |
| 45 | should navigate to edit page when Edit is clicked in context menu | Open context menu | Click "Edit" | URL changes to `/products/{id}/edit` |
| 46 | should navigate to create page with source_product_id when Duplicate is clicked | Open context menu on product | Click "Duplicate" | URL changes to `/products/create?source_product_id={id}` |
| 47 | should show delete confirmation dialog when Delete is clicked in context menu | Open context menu | Click "Delete" | Dialog visible: "Delete Product?" with product name, status, SKU count. "Cancel" and "Delete Product" buttons |
| 48 | should delete product when confirmed in delete dialog | Delete dialog open | Click "Delete Product" | API call: DELETE /seller-products/{id}. Toast: "Product deleted". Product disappears from list. Status counts update |
| 49 | should not delete product when cancel is clicked in delete dialog | Delete dialog open | Click "Cancel" | Dialog closes. Product remains in list unchanged |
| 50 | should deactivate product via context menu single status change | Open context menu on active product | Click "Deactivate" | API: POST /bulk-status with single ID, status=inactive. Toast: "Product deactivated". Product moves to inactive |
| 51 | should activate product via context menu | Open context menu on inactive product | Click "Activate" | API: POST /bulk-status, status=active. Toast: "Product activated" |
| 52 | should show only Edit and Duplicate for pending products | Open context menu on pending product | Observe menu items | Only Edit and Duplicate available. No Activate/Deactivate/Delete |
| 53 | should show only Edit and Duplicate for violation products | Open context menu on violation product | Observe menu | Only Edit and Duplicate. Violation icon visible on row with tooltip showing reason |
| 54 | should show only View for deleted products | Click context menu on deleted product (Deleted tab) | Observe menu | Only "View" option. No edit/delete/status actions |

### Bulk Operations (`product-list-bulk.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 55 | should show bulk action bar when one or more products are selected | Products in list | Check one product checkbox | Bulk action bar appears at bottom: "1 product selected" with action buttons |
| 56 | should select all products on current page when header checkbox is clicked | Products in list | Click header checkbox | All row checkboxes checked. Selected count shows total page count. Bulk bar visible |
| 57 | should deselect all when header checkbox is clicked again | All selected | Click header checkbox | All row checkboxes unchecked. Bulk bar disappears |
| 58 | should show correct count in bulk bar as products are selected/deselected | Select 3 products | Observe bulk bar | "3 products selected" text in bulk bar |
| 59 | should hide bulk action bar when cancel selection is clicked | Products selected, bulk bar visible | Click "Cancel Selection" | All checkboxes unchecked. Bulk bar disappears |
| 60 | should show available bulk actions based on current tab | On Active tab, select products | Observe bulk bar buttons | "Deactivate" and "Delete" visible. "Activate" NOT visible (already active) |
| 61 | should show Activate button on Inactive tab | Switch to Inactive tab, select products | Observe bulk bar | "Activate" and "Delete" visible. "Deactivate" NOT visible |
| 62 | should show only Delete on Draft tab | Switch to Draft tab, select products | Observe bulk bar | Only "Delete" visible |
| 63 | should show confirmation dialog before bulk deactivate | Select 3 active products | Click "Deactivate" in bulk bar | Dialog: "Deactivate 3 Products?" listing product names. "Cancel" and "Deactivate" buttons |
| 64 | should deactivate selected products when confirmed | Bulk deactivate dialog open, 3 products selected | Click "Deactivate" | API: POST /bulk-status { product_ids: [...], status: "inactive" }. Toast: "3 products deactivated". Selection cleared. Products disappear from Active tab. Status counts update |
| 65 | should show confirmation dialog before bulk delete | Select 3 products | Click "Delete" in bulk bar | Dialog: "Delete 3 Products?" listing names. Warning about deletion being permanent |
| 66 | should delete selected products when confirmed | Bulk delete dialog open | Click "Delete Products" | API: POST /bulk-delete. Toast: "3 products deleted". Selection cleared. Products moved to Deleted |
| 67 | should handle partial success in bulk operation | Select products with mixed statuses (draft + active) | Bulk deactivate | Toast: "2 products deactivated. 1 skipped (invalid status)." with expandable details showing skipped product and reason |
| 68 | should handle bulk operation failure (403) | Select products, server returns 403 | Confirm bulk action | Toast: "Failed to update products. Some products not found or unauthorized." Bulk bar stays visible. Selection preserved for retry |
| 69 | should batch requests when selection exceeds 50 items | Set per_page to 100, select 60 products | Bulk deactivate, confirm | Two sequential API calls (50 + 10). Aggregated toast: "60 products deactivated" |
| 70 | should activate selected products via bulk activate | On Inactive tab, select 2 products | Click "Activate", confirm | API: POST /bulk-status { status: "active" }. Toast: "2 products activated". Products move to Active tab |

### Bulk Operation Server Errors

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 71 | should show error when bulk status called with empty product_ids | Intercept to return 422 "product_ids field is required" | Attempt bulk action with no selection (edge) | Toast: "No products selected." Selection state unchanged |
| 72 | should show error when bulk status with invalid target status | Intercept to return 422 "status must be one of: active, inactive, deleted" | Confirm bulk action | Toast: "Invalid status for bulk operation." |
| 73 | should show error when bulk delete with product_ids exceeding 50 | Select 51 products (per_page=100 scenario) | Confirm bulk delete | Application batches in groups of 50. If server rejects: error toast |
| 74 | should show error for invalid product ID in bulk request | Intercept 422 "product_ids.0 must be a valid integer" | Confirm bulk action | Toast: "Invalid product selection. Please refresh and try again." |

### Loading States (`product-list-loading.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 75 | should show skeleton rows while products are loading | Navigate to list (first load) | Observe during load | 5 skeleton rows visible matching table layout (checkbox, image rect, title bar, status pill, price bar, SKU count). Shimmer animation active |
| 76 | should show skeleton badges on status tabs while counts are loading | Navigate to list | Observe tabs during load | Tab badges show skeleton placeholders until status-counts API responds |
| 77 | should show background refetch indicator when re-fetching with existing data | Products already loaded | Change a filter (triggers refetch) | Thin progress bar at top. Existing data visible at reduced opacity. No skeleton rows |

### Error States (`product-list-errors.spec.ts`)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 78 | should show error state when product list API fails | Intercept API to return network error | Navigate to list | Error state: "Failed to load products" with "Check your connection and try again" and "Retry" button |
| 79 | should reload products when retry button is clicked | Error state showing | Click "Retry" | API call re-fires. On success, products displayed. Error state disappears |
| 80 | should handle invalid URL parameters gracefully | Navigate with `?sort_by=invalid&per_page=500` | Observe | Invalid params stripped from URL. Toast: "Invalid filter removed. Showing default results." Default results shown |
| 81 | should show toast for invalid status URL parameter | Navigate with `?status=archived` | Observe | URL corrected (status param removed). Toast shown. Products loaded with default (all) filter |
| 82 | should show toast for non-UUID shipping_category_id | Navigate with `?shipping_category_id=not-a-uuid` | Observe | URL corrected. Toast shown. Products loaded without category filter |
| 83 | should show toast for non-integer region_id | Navigate with `?region_id=abc` | Observe | URL corrected. Toast shown. Products loaded without region filter |
| 84 | should show error when per_page exceeds maximum of 100 | Navigate with `?per_page=500` | Observe | URL corrected to default per_page. Toast: "Invalid filter removed." Products load with default pagination |

### Table Row Details

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 85 | should show correct status badge color for each status | Create products in each status | Navigate to list | Active=green, Inactive=gray, Draft=yellow, Pending=blue, Violation=red, Deleted=dark gray |
| 86 | should show status tooltip on badge hover | Product with active status | Hover status badge | Tooltip shows: "Live on storefront" with "Updated X ago" |
| 87 | should show violation reason in tooltip for violation products | Create violation product | Hover violation badge | Tooltip shows violation reason text |
| 88 | should show context menu on row hover | Product row visible | Hover over row | Row background changes to gray-50. Three-dot context menu icon appears |
| 89 | should highlight row when checkbox is selected | Select a product | Observe row | Row background changes to blue-50. Checkbox shows filled check mark |

### Accessibility

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 90 | should have role=tablist on status tab container | Navigate to list | Query tab container | Container has `role="tablist"` |
| 91 | should have aria-selected on active tab | Click Active tab | Query tab attributes | Active tab has `aria-selected="true"`. Others have `aria-selected="false"` |
| 92 | should have proper table structure with thead and tbody | Navigate with products | Query table | Table has `<thead>` with column headers using `scope="col"`. Body rows in `<tbody>` |
| 93 | should have aria-label on header checkbox | Navigate to list | Query header checkbox | `aria-label="Select all products on this page"` |
| 94 | should have aria-label on row checkboxes with product title | Products in list | Query first row checkbox | `aria-label="Select {product title}"` |
| 95 | should have aria-current=page on current pagination button | On page 2 | Query page 2 button | `aria-current="page"` attribute present |
| 96 | should announce filter results via aria-live region | Change a filter | Observe after results load | `aria-live="polite"` region announces "Showing {n} products" |
