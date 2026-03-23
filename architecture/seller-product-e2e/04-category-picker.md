# Category Picker Component — E2E Test Architecture

## Page Object

### CategoryPickerComponent

```
File: e2e/pages/category-picker.component.ts
Class: CategoryPickerComponent
```

**Constructor parameter:** `page: Page`

**Locators:**

| Locator | Selector | Description |
|---------|----------|-------------|
| `categoryField` | `page.getByRole('combobox', { name: /shipping category/i })` | Category selector field |
| `browseButton` | `page.getByRole('button', { name: /browse/i })` | Opens category tree modal |
| `clearButton` | `page.getByRole('button', { name: /clear category/i })` | Clears selected category (x button) |
| `modal` | `page.getByRole('dialog', { name: /select shipping category/i })` | Category picker modal |
| `modalSearchInput` | `modal.getByRole('searchbox', { name: /search categories/i })` | Search input inside modal |
| `categoryTree` | `modal.getByRole('tree')` | Tree container |
| `selectedDisplay` | `page.getByTestId('selected-category-path')` | Displays selected category path |
| `confirmButton` | `modal.getByRole('button', { name: /select category/i })` | Confirms selection in modal |
| `cancelButton` | `modal.getByRole('button', { name: /cancel/i })` | Cancels modal |
| `recentList` | `page.getByTestId('recent-categories')` | Recently used categories panel |
| `changeWarningDialog` | `page.getByRole('alertdialog', { name: /change category/i })` | Category change warning dialog |
| `noResultsMessage` | `modal.getByText(/no categories match/i)` | No search results message |

**Actions:**

| Method | Description |
|--------|-------------|
| `async openBrowseModal()` | Clicks browse button, waits for modal visible |
| `async searchCategory(query: string)` | Types query in modal search input |
| `async clearSearch()` | Clears search input in modal |
| `async expandParent(name: string)` | Clicks a parent node to expand children |
| `async selectLeafCategory(name: string)` | Clicks a leaf node in the tree |
| `async confirmSelection()` | Clicks confirm button in modal |
| `async cancelModal()` | Clicks cancel button in modal |
| `async clearCategory()` | Clicks the clear (x) button on selected category |
| `async selectFromRecent(categoryPath: string)` | Clicks a recent category entry |
| `async confirmCategoryChange()` | Confirms category change warning dialog |
| `async cancelCategoryChange()` | Cancels category change warning dialog |
| `getTreeItem(name: string): Locator` | Returns a tree item locator by name |
| `getRecentItem(path: string): Locator` | Returns a recent category item locator |

---

## Helpers

Uses `productTestData.categoryId()` and `productTestData.noSchemaCategoryId()` from `product-data.helper.ts` for known category UUIDs.

Uses `PRODUCT_CONSTANTS.CATEGORY_WITH_SCHEMA` and `PRODUCT_CONSTANTS.CATEGORY_NO_SCHEMA` for category metadata (path, attributes).

---

## Fixture Wiring

```typescript
categoryPickerComponent: async ({ page }, use) => {
  await use(new CategoryPickerComponent(page))
}
```

---

## Spec File Breakdown

| Spec File | Concern |
|-----------|---------|
| `product-create-category.spec.ts` | Category picker in create context |
| `product-edit-page.spec.ts` | Category display and change in edit context (subset of edit tests) |

Category picker tests are part of the create flow spec files since the picker is embedded in the create page. Edit flow category tests are in the edit page spec.

---

## Test Scenarios

### Browse Mode

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 1 | should open category tree modal when browse button is clicked | Navigate to create page | Click browse button | Modal is visible with search input and tree |
| 2 | should display top-level categories in tree | Open browse modal | Observe tree | Top-level parent nodes visible (Electronics, Apparel, Health, etc.) |
| 3 | should expand parent node to show children when clicked | Open browse modal | Click a parent node (e.g., "Electronics & Gadgets") | Children become visible (Computer Peripherals, Smart Home, etc.) |
| 4 | should collapse expanded parent node when clicked again | Expand a parent node | Click same parent node | Children are hidden, chevron rotates back |
| 5 | should only allow leaf nodes to be selected | Open browse modal, expand to leaf level | Click a leaf node (e.g., "Mouse") | Leaf shows selected state (check icon, blue bg). Parent nodes cannot be selected — clicking them only expands/collapses |
| 6 | should not allow selecting a non-leaf (parent) node as category | Open browse modal | Click a parent node (e.g., "Computer Peripherals") | Node expands children but is NOT selected as category. Confirm button remains disabled |
| 7 | should show selected category path at bottom of modal | Open modal, select a leaf | Observe footer area | Path displayed: "Electronics & Gadgets > Computer Peripherals > Mouse" |
| 8 | should close modal and populate field when confirm is clicked | Select a leaf, click confirm | Observe form | Modal closes. Category field shows full path. Form sections below unlock |
| 9 | should not change category when cancel is clicked | Select a leaf in modal, click cancel | Observe form | Modal closes. Category field unchanged. Form sections remain in previous state |
| 10 | should close modal when X button is clicked | Open modal | Click X close button | Modal closes without changing selection |

### Search Mode

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 11 | should filter tree nodes when typing in search input | Open browse modal | Type "mouse" in search | Only categories matching "mouse" are visible. Non-matching branches hidden |
| 12 | should show leaf nodes with full path when search matches | Open modal | Search for a leaf name | Matching leaf nodes visible with their parent path |
| 13 | should show no results message when search has no matches | Open modal | Type "xyznonexistent" | "No categories match your search" message visible |
| 14 | should clear search and restore full tree when search is cleared | Search for "mouse" (filtered results showing) | Clear search input | Full tree restored with all categories visible |
| 15 | should allow selecting a leaf from search results | Search for a leaf, find it in filtered results | Click the leaf node | Leaf is selected, path shown at bottom, confirm button enabled |

### Recently Used Categories

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 16 | should display recently used categories panel when history exists | Set localStorage `recent-categories` with entries, navigate to create | Observe category section | Recently used panel visible with up to 5 entries, most recent first |
| 17 | should not display recently used panel for first-time user | Clear localStorage, navigate to create | Observe category section | No recently used panel visible. Only browse button shown |
| 18 | should select category directly from recently used list | Recently used panel visible | Click "Select" on a recent entry | Category field populated with that entry. Form sections unlock. No modal opened |
| 19 | should update recently used list after new category selection | Select a category via browse modal | Check localStorage | New category appears at top of recent-categories list. List limited to 5 entries |
| 20 | should deduplicate recently used categories by id | Select same category twice via browse | Check localStorage | Only one entry for that category in list (updated timestamp) |

### Selection Display and Clearing

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 21 | should display selected category as full breadcrumb path | Select a leaf category | Observe category field | Full path shown: "Level 1 > Level 2 > Leaf" with clear (x) button |
| 22 | should clear category and show warning when clear button is clicked with existing data | Select a category, add properties and SKUs | Click clear (x) on category | Change warning dialog appears listing what will be reset |
| 23 | should reset properties, SKUs, regional pricing, and specs when category change is confirmed | Category selected with properties/SKUs/specs | Click clear, confirm change | All variation properties removed. SKU matrix cleared. Specs reset. Regional pricing cleared. Title and description preserved. Gallery preserved |
| 24 | should keep current category when change warning is cancelled | Category selected with data | Click clear, cancel warning | Category unchanged. All form data preserved |

### Locked State (Edit Context)

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 25 | should display current category in edit mode | Navigate to edit page for product with category | Observe category section | Category field shows full path from loaded product |
| 26 | should show category change warning in edit mode when clearing | In edit mode with existing SKUs | Click clear on category | Warning dialog appears explaining cascade effects including SKU deletion |

### Disabled State

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 27 | should disable all form sections below until category is selected | Navigate to create page (no category) | Observe form sections | Basic Info, Variations, Specs, Gallery, Regional Pricing sections are disabled/collapsed with hint "Select a category first" |
| 28 | should enable all form sections after category selection | Navigate to create, select a category | Observe form sections | All sections become editable. Schema-driven fields populated from category |

### Schema Loading

| # | Scenario | Arrange | Act | Assert |
|---|----------|---------|-----|--------|
| 29 | should load category schema variation attributes after selection | Select a category with schema | Observe variations section | Property dropdowns populated with schema-defined properties (e.g., Color, DPI) |
| 30 | should load category schema specification attributes after selection | Select a category with schema | Observe specifications section | Required specs marked with asterisk. Select-type specs show dropdown with schema options |
| 31 | should show free-form fields when category has no schema | Select a category without schema | Observe variations and specs sections | Property names are free-text inputs. Spec names and values are free-text. No required markers |
