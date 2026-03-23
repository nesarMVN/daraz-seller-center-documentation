# Cross-Cutting Concerns

This document defines the shared infrastructure, test data strategy, authentication setup, cleanup strategy, and parallel safety rules that apply across all Seller Product E2E tests.

---

## Authentication Setup

### Storage State Reuse

All product tests require an authenticated seller session. The auth setup project runs first and saves `storageState` to a JSON file. Product test specs use this state to skip login on every test.

```
e2e/playwright.config.ts projects:
  1. "auth-setup" — logs in, saves storageState to .auth/user.json
  2. "seller-product" — depends on "auth-setup", uses storageState

storageState path: .auth/user.json
```

### Auth Helper

```
File: e2e/helpers/auth.helper.ts

export const authHelper = {
  /**
   * Performs login via UI and saves storage state.
   * Called only by the auth-setup project.
   */
  async loginAndSaveState(page: Page, path: string): Promise<void>

  /**
   * Returns test seller credentials from environment variables.
   * E2E_SELLER_EMAIL and E2E_SELLER_PASSWORD must be set.
   */
  credentials(): { email: string; password: string }
}
```

### Environment Variables

| Variable | Purpose |
|----------|---------|
| `E2E_SELLER_EMAIL` | Seller account email for E2E tests |
| `E2E_SELLER_PASSWORD` | Seller account password |
| `E2E_BASE_URL` | Base URL (default: `http://localhost:3846`) |

---

## Test Data Strategy

### API-Based Test Data Creation

Product tests that need existing products (list page, edit flow) should create test data via direct API calls in `beforeAll` or `beforeEach`, not through the UI. This is faster, more reliable, and isolates tests from create-flow bugs.

```
File: e2e/helpers/product-api.helper.ts

export const productApiHelper = {
  /**
   * Creates a product via POST /seller-products.
   * Returns the full product response including id and etag.
   */
  async createProduct(
    request: APIRequestContext,
    data: Partial<CreateProductPayload>
  ): Promise<{ id: number; etag: string }>

  /**
   * Creates a minimal draft product (category + title only).
   */
  async createDraftProduct(
    request: APIRequestContext
  ): Promise<{ id: number; etag: string }>

  /**
   * Creates a product with full data (properties, SKUs, specs, gallery).
   * Status defaults to "active" via admin approval simulation.
   */
  async createFullProduct(
    request: APIRequestContext,
    overrides?: Partial<CreateProductPayload>
  ): Promise<{ id: number; etag: string }>

  /**
   * Creates multiple products for list page tests.
   * Returns array of { id, title, status }.
   */
  async createProductBatch(
    request: APIRequestContext,
    count: number,
    statusDistribution?: Record<string, number>
  ): Promise<Array<{ id: number; title: string; status: string }>>

  /**
   * Deletes a product by ID (soft delete via API).
   */
  async deleteProduct(
    request: APIRequestContext,
    id: number
  ): Promise<void>

  /**
   * Fetches a product by ID with all expansions.
   */
  async getProduct(
    request: APIRequestContext,
    id: number
  ): Promise<FullProductResponse>

  /**
   * Updates product status via bulk-status endpoint.
   */
  async setProductStatus(
    request: APIRequestContext,
    id: number,
    status: 'active' | 'inactive' | 'deleted'
  ): Promise<void>
}
```

### Test Data Generation

```
File: e2e/helpers/product-data.helper.ts

export const productTestData = {
  /**
   * Generates a unique product title with timestamp suffix.
   * Pattern: "E2E Test Product {faker word} {timestamp}"
   */
  title(): string

  /**
   * Generates a valid HTML description.
   */
  description(): string

  /**
   * Returns a known leaf category UUID from the test environment.
   * Uses a category that has a schema with variation_attributes
   * and specification_attributes defined.
   */
  categoryId(): string

  /**
   * Returns a category UUID for a category with NO schema.
   */
  noSchemaCategoryId(): string

  /**
   * Generates property data for create requests.
   * count: number of properties (1-3)
   * valuesPerProperty: number of values per property (1-50)
   */
  properties(count?: number, valuesPerProperty?: number): PropertyPayload[]

  /**
   * Generates SKU data matching the given properties.
   * Creates cartesian product of property values.
   */
  skus(properties: PropertyPayload[]): SkuPayload[]

  /**
   * Generates specification data.
   */
  specifications(count?: number): SpecPayload[]

  /**
   * Generates gallery data with one primary image.
   */
  gallery(imageCount?: number, videoCount?: number): GalleryPayload[]

  /**
   * Generates regional price data for a SKU.
   */
  regionalPrices(regionIds?: number[]): RegionalPricePayload[]

  /**
   * Returns a complete valid product payload ready for POST.
   */
  fullProduct(overrides?: Partial<CreateProductPayload>): CreateProductPayload

  /**
   * Returns a minimal draft payload (category + title only).
   */
  draftProduct(): CreateProductPayload
}
```

### Known Test Data Constants

```
File: e2e/helpers/product-constants.helper.ts

export const PRODUCT_CONSTANTS = {
  /** Known leaf category with schema (variation + spec attributes) */
  CATEGORY_WITH_SCHEMA: {
    id: 'uuid-from-env-or-seeded',
    path: 'Electronics & Gadgets > Computer Peripherals > Mouse',
    variationAttributes: ['Color', 'DPI'],
    requiredSpecs: ['Brand', 'Connectivity'],
    optionalSpecs: ['Sensor Type', 'DPI Range', 'Weight'],
  },

  /** Known leaf category with NO schema */
  CATEGORY_NO_SCHEMA: {
    id: 'uuid-from-env-or-seeded',
    path: 'General > Uncategorized > Other',
  },

  /** Region IDs for regional pricing tests */
  REGIONS: {
    BD: 1,
    IN: 2,
    AE: 3,
    US: 4,
    EU: 5,
    AU: 6,
  },

  /** Valid product statuses */
  STATUSES: ['draft', 'pending', 'active', 'inactive', 'violation', 'deleted'] as const,

  /** Max limits from validation rules */
  LIMITS: {
    TITLE_MAX: 255,
    DESCRIPTION_MAX: 65535,
    PROPERTIES_MAX: 3,
    VALUES_PER_PROPERTY_MAX: 50,
    SKUS_MAX: 100,
    SPECIFICATIONS_MAX: 50,
    GALLERY_MAX: 20,
    PRICE_MIN: 0.01,
    PRICE_MAX: 9999999999.99,
    SKU_CODE_MAX: 50,
    PROPERTY_NAME_MAX: 100,
    PROPERTY_VALUE_MAX: 100,
    SPEC_NAME_MAX: 100,
    SPEC_VALUE_MAX: 500,
    GALLERY_URL_MAX: 500,
    BULK_IDS_MAX: 50,
    PER_PAGE_MAX: 100,
  },

  /** Sort fields for list page */
  SORT_FIELDS: ['title', 'created_at', 'updated_at', 'price'] as const,

  /** Sort orders */
  SORT_ORDERS: ['asc', 'desc'] as const,

  /** Expand options for show endpoint */
  EXPAND_OPTIONS: ['properties', 'skus', 'specifications', 'gallery', 'category', 'regional_prices'] as const,
}
```

---

## Cleanup Strategy

### Per-Test Cleanup

Each test that creates products via API should clean up after itself in `afterEach` or `afterAll`. Use soft delete via the API.

```typescript
test.afterEach(async ({ request }) => {
  for (const id of createdProductIds) {
    await productApiHelper.deleteProduct(request, id).catch(() => {})
  }
  createdProductIds = []
})
```

### Batch Cleanup

For list page tests that create many products, clean up the entire batch in `afterAll`:

```typescript
let createdIds: number[] = []

test.beforeAll(async ({ request }) => {
  const products = await productApiHelper.createProductBatch(request, 25)
  createdIds = products.map(p => p.id)
})

test.afterAll(async ({ request }) => {
  if (createdIds.length > 0) {
    await productApiHelper.bulkDelete(request, createdIds)
  }
})
```

### Orphaned Data

If a test fails before cleanup, orphaned products accumulate. Two strategies:

1. **Title prefix convention**: All E2E products use the title prefix `"E2E Test Product"`. A nightly cleanup job can delete products matching this prefix.
2. **Timestamp-based TTL**: Products created more than 24 hours ago with the E2E prefix are considered orphaned and safe to delete.

---

## Parallel Safety

### Unique Data per Test

Every test must use unique data to avoid collisions when tests run in parallel (Playwright workers):

- **Titles**: Include `faker` random words and timestamps. Never use hardcoded titles like "Test Product".
- **SKU codes**: Include random suffix. e.g., `"BK-800-{nanoid}"`.
- **Category**: All tests can share the same category since it is read-only reference data.

### No Shared Mutable State

- Tests must NOT depend on a specific product count in the database.
- Tests must NOT depend on a specific order of products in the list (unless they set up their own data and filter by title).
- Tests that check "empty state" should filter by a unique search term that returns zero results, not rely on a globally empty database.

### Worker-Scoped Fixtures

If multiple test files need the same set-up data (e.g., a product with full SKUs for edit tests), use worker-scoped fixtures so the data is created once per worker, not once per test:

```typescript
// In fixtures
productForEdit: [
  async ({ request }, use) => {
    const product = await productApiHelper.createFullProduct(request)
    await use(product)
    await productApiHelper.deleteProduct(request, product.id).catch(() => {})
  },
  { scope: 'worker' }
]
```

### Parallel-Hostile Tests

Some tests cannot safely run in parallel:

- **Bulk operations on the list page**: Tests that select "all" products on a page could interfere with each other. These tests should create their own isolated product set and filter by a unique title prefix.
- **Status count tests**: Tests that verify badge counts must create their own data set and verify counts relative to that data (not absolute counts).

---

## Fixture Wiring

### Product Fixtures Addition

```
File addition to: e2e/fixtures/index.ts

New fixtures to add:

type ProductFixtures = {
  productListPage: ProductListPage
  productCreatePage: ProductCreatePage
  productEditPage: ProductEditPage
  categoryPickerComponent: CategoryPickerComponent
  galleryManagerComponent: GalleryManagerComponent
  skuBuilderComponent: SkuBuilderComponent
  regionalPricingComponent: RegionalPricingComponent
}

Each fixture instantiates the page object with the test's `page` instance:

productListPage: async ({ page }, use) => {
  await use(new ProductListPage(page))
}
```

### Shared Helper Fixtures

```
type HelperFixtures = {
  productApi: typeof productApiHelper
  productData: typeof productTestData
}

productApi: async ({ request }, use) => {
  await use(productApiHelper)
}

productData: async ({}, use) => {
  await use(productTestData)
}
```

---

## Test Organization

### Folder Structure

```
e2e/tests/seller-product/
  product-list/
    product-list-page.spec.ts           # Page rendering, layout, elements
    product-list-tabs.spec.ts           # Status tab navigation and counts
    product-list-search.spec.ts         # Search input, debounce, clear
    product-list-filters.spec.ts        # Category filter, region filter, sorting
    product-list-pagination.spec.ts     # Pagination controls, page navigation
    product-list-empty-states.spec.ts   # All empty state variations
    product-list-actions.spec.ts        # Single product actions (edit, delete, status, duplicate)
    product-list-bulk.spec.ts           # Bulk selection and bulk operations
    product-list-loading.spec.ts        # Loading skeletons, background refetch
    product-list-errors.spec.ts         # Error states (network, 422, bulk errors)
  product-create/
    product-create-page.spec.ts         # Page layout, section presence
    product-create-category.spec.ts     # Category picker in create context
    product-create-basic-info.spec.ts   # Title and description fields
    product-create-variations.spec.ts   # Property and SKU builder in create
    product-create-specs.spec.ts        # Specification form in create
    product-create-gallery.spec.ts      # Gallery manager in create context
    product-create-regional.spec.ts     # Regional pricing in create context
    product-create-draft.spec.ts        # Save as draft workflow
    product-create-submit.spec.ts       # Submit for review workflow
    product-create-validation.spec.ts   # All validation error scenarios
    product-create-source.spec.ts       # Source product pre-fill flow
    product-create-autosave.spec.ts     # Auto-save and recovery
  product-edit/
    product-edit-page.spec.ts           # Page layout, data loading, pre-fill
    product-edit-changes.spec.ts        # Change tracking, dirty state
    product-edit-save.spec.ts           # Save changes workflow
    product-edit-conflict.spec.ts       # ETag conflict resolution (412)
    product-edit-validation.spec.ts     # Edit-specific validation errors
    product-edit-status.spec.ts         # Status transitions from edit page
    product-edit-cancel.spec.ts         # Cancel and unsaved changes warning
    product-edit-violation.spec.ts      # Violation resubmission workflow
    product-edit-errors.spec.ts         # All PATCH error scenarios
    product-edit-delete.spec.ts         # Delete from edit page

e2e/pages/
  product-list.page.ts
  product-create.page.ts
  product-edit.page.ts
  category-picker.component.ts
  gallery-manager.component.ts
  sku-builder.component.ts
  regional-pricing.component.ts

e2e/helpers/
  product-api.helper.ts
  product-data.helper.ts
  product-constants.helper.ts
```

### Naming Convention

- Spec files: `{feature}-{concern}.spec.ts` (kebab-case)
- Page objects: `{feature}.page.ts` or `{component}.component.ts`
- Helpers: `{feature}-{purpose}.helper.ts`
- Test names: `should {expected behavior}` (lowercase, no period)

### Max 200 Lines per Spec

Each spec file should stay under 200 lines. If a concern has more than ~15 tests, split it into sub-concerns (e.g., `product-list-bulk-select.spec.ts` and `product-list-bulk-actions.spec.ts`).

---

## Import Conventions

All E2E imports use the `@e2e/*` path alias:

```typescript
import { test, expect } from '@e2e/fixtures'
import { ProductListPage } from '@e2e/pages/product-list.page'
import { productTestData } from '@e2e/helpers/product-data.helper'
import { PRODUCT_CONSTANTS } from '@e2e/helpers/product-constants.helper'
```

Never use relative imports (`../../../pages/...`).

---

## Selector Priority

All locators in page objects follow this priority:

1. `getByRole` — buttons, links, headings, tabs, checkboxes, textboxes, comboboxes
2. `getByLabel` — form inputs with labels
3. `getByText` — visible text content
4. `getByTestId` — last resort for elements without accessible roles or labels

Example:
```typescript
// Good
this.newProductButton = page.getByRole('button', { name: /new product/i })
this.searchInput = page.getByRole('searchbox', { name: /search products/i })
this.activeTab = page.getByRole('tab', { name: /active/i })

// Acceptable when no role applies
this.productTitle = page.getByText('Product Title')

// Last resort
this.skuMatrix = page.getByTestId('sku-matrix-table')
```

---

## Assertion Rules

- Assertions live in spec files, never in page objects.
- Page objects return locators or perform actions; specs assert on results.
- Use `toBeVisible()`, `toHaveText()`, `toHaveValue()`, `toHaveURL()`, `toHaveCount()`.
- Avoid `waitForTimeout()`. Use `waitForResponse()`, `waitForURL()`, or auto-waiting locators.
- For API response assertions, use `page.waitForResponse()` to intercept and validate.

---

## Network Interception for Error Tests

Tests that verify error states (network errors, 422 responses) should use `page.route()` to intercept API calls and return mock error responses. This is the one exception to "no mocking" — we mock network failures, not business logic.

```typescript
// Simulate network error
await page.route('**/seller-products**', route => route.abort())

// Simulate 422 response
await page.route('**/seller-products', route =>
  route.fulfill({
    status: 422,
    contentType: 'application/json',
    body: JSON.stringify({
      message: 'The given data was invalid.',
      errors: { title: ['The title field is required.'] }
    })
  })
)
```

Note: `page.route()` intercepts at the network layer, which is different from mocking application code. The application still makes real requests; we just intercept them at the transport level to simulate server behavior that would be difficult to trigger naturally (like 500 errors or specific 422 combinations).

---

## Error Interception Patterns by Endpoint

Different endpoints require different interception strategies. This section documents the `page.route()` patterns needed for each error category.

### List Endpoint Errors (GET /seller-products)

```typescript
// Simulate 422 for invalid query parameters
await page.route('**/seller-products?**', route =>
  route.fulfill({
    status: 422,
    contentType: 'application/json',
    body: JSON.stringify({
      message: 'The given data was invalid.',
      errors: { status: ['The status must be one of: draft, pending, active, inactive, violation, deleted.'] }
    })
  })
)

// Simulate network timeout
await page.route('**/seller-products?**', route => route.abort('timedout'))
```

### Create Endpoint Errors (POST /seller-products)

```typescript
// Simulate 422 with field-level errors
await page.route('**/seller-products', route => {
  if (route.request().method() === 'POST') {
    return route.fulfill({
      status: 422,
      contentType: 'application/json',
      body: JSON.stringify({
        message: 'The given data was invalid.',
        errors: {
          title: ['The title field is required.'],
          'properties.0.name': ["The property 'Weight' is not defined in the category schema."],
          'skus.2.price': ['The skus.2.price must be at least 0.01.']
        }
      })
    })
  }
  return route.continue()
})

// Simulate 400 transaction failure
await page.route('**/seller-products', route => {
  if (route.request().method() === 'POST') {
    return route.fulfill({
      status: 400,
      contentType: 'application/json',
      body: JSON.stringify({ message: 'Failed to create product. Please try again.' })
    })
  }
  return route.continue()
})
```

### Update Endpoint Errors (PATCH /seller-products/{id})

```typescript
// Simulate 412 ETag conflict
await page.route('**/seller-products/*', route => {
  if (route.request().method() === 'PATCH') {
    return route.fulfill({
      status: 412,
      contentType: 'application/json',
      body: JSON.stringify({
        message: 'Precondition failed. The resource has been modified. Re-fetch and retry.'
      })
    })
  }
  return route.continue()
})

// Simulate 415 wrong Content-Type
await page.route('**/seller-products/*', route => {
  if (route.request().method() === 'PATCH') {
    return route.fulfill({
      status: 415,
      contentType: 'application/json',
      body: JSON.stringify({
        message: 'Unsupported media type. Expected: application/json-patch+json.'
      })
    })
  }
  return route.continue()
})

// Simulate 409 test operation failure
await page.route('**/seller-products/*', route => {
  if (route.request().method() === 'PATCH') {
    return route.fulfill({
      status: 409,
      contentType: 'application/json',
      body: JSON.stringify({
        message: "Test failed at '/title': expected 'Old Title', got 'Changed Title'."
      })
    })
  }
  return route.continue()
})
```

### Bulk Endpoint Errors (POST /seller-products/bulk-status, /bulk-delete)

```typescript
// Simulate 403 partial authorization failure
await page.route('**/seller-products/bulk-status', route =>
  route.fulfill({
    status: 403,
    contentType: 'application/json',
    body: JSON.stringify({
      message: 'Some products not found or unauthorized.'
    })
  })
)

// Simulate 422 validation on bulk
await page.route('**/seller-products/bulk-delete', route =>
  route.fulfill({
    status: 422,
    contentType: 'application/json',
    body: JSON.stringify({
      message: 'The given data was invalid.',
      errors: { product_ids: ['The product_ids must not have more than 50 items.'] }
    })
  })
)
```

---

## Response Interception for Assertions

To verify that the frontend sends correct data (e.g., correct JSON Patch operations), intercept outgoing requests with `page.waitForResponse()`:

```typescript
// Capture and verify the PATCH request body
const patchPromise = page.waitForResponse(
  response => response.url().includes('/seller-products/') && response.request().method() === 'PATCH'
)
await productEditPage.clickSaveChanges()
const patchResponse = await patchPromise
const requestBody = patchResponse.request().postDataJSON()

// Assert specific operations in the patch
expect(requestBody).toContainEqual(
  expect.objectContaining({ op: 'replace', path: '/title' })
)

// Verify If-Match header was sent
const ifMatch = patchResponse.request().headers()['if-match']
expect(ifMatch).toBeTruthy()
```

This pattern is used in save, status transition, and conflict resolution tests to verify the frontend generates correct JSON Patch operations without needing to inspect server-side state.
