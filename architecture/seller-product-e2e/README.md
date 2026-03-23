# Seller Product E2E Test Architecture

E2E test plan for the Seller Product feature. Each document defines page objects, helpers, fixture wiring, spec file breakdown, and every testable scenario as a `should ...` statement with arrange/act/assert descriptions.

## Documents

| # | File | Scope | Scenarios |
|---|------|-------|-----------|
| 1 | [01-product-list-page.md](./01-product-list-page.md) | Product list page: rendering, tabs, search, filters, sorting, pagination, empty states, single actions, bulk operations, bulk errors, loading, errors, accessibility | 96 |
| 2 | [02-product-create-flow.md](./02-product-create-flow.md) | Product create flow: page layout, basic info, variations/SKUs, specifications, regional pricing, save as draft, submit for review, all validation errors, source product pre-fill, auto-save | 109 |
| 3 | [03-product-edit-flow.md](./03-product-edit-flow.md) | Product edit flow: data loading, change tracking, save (JSON Patch), ETag conflict (412), validation errors, status transitions, cancel/unsaved changes, violation resubmission, all PATCH errors (document, collection, gallery, regional), delete, category change | 108 |
| 4 | [04-category-picker.md](./04-category-picker.md) | Category picker component: browse mode, search mode, recently used, selection display, locked state, disabled state, schema loading | 31 |
| 5 | [05-gallery-manager.md](./05-gallery-manager.md) | Gallery manager component: empty state, add images/videos, drag-and-drop reorder, set primary, remove items, limit enforcement, validation, card states, error recovery | 32 |
| 6 | [06-cross-cutting-concerns.md](./06-cross-cutting-concerns.md) | Auth setup, test data strategy, cleanup, parallel safety, fixture wiring, folder structure, import conventions, selector priority, assertion rules | — |

**Total test scenarios: 376**

## Architecture Summary

### Page Objects

| Class | File | Route/Component |
|-------|------|-----------------|
| `ProductListPage` | `e2e/pages/product-list.page.ts` | `/products` |
| `ProductCreatePage` | `e2e/pages/product-create.page.ts` | `/products/create` |
| `ProductEditPage` | `e2e/pages/product-edit.page.ts` | `/products/{id}/edit` |
| `CategoryPickerComponent` | `e2e/pages/category-picker.component.ts` | Embedded in create/edit |
| `GalleryManagerComponent` | `e2e/pages/gallery-manager.component.ts` | Embedded in create/edit |
| `SkuBuilderComponent` | `e2e/pages/sku-builder.component.ts` | Embedded in create/edit |
| `RegionalPricingComponent` | `e2e/pages/regional-pricing.component.ts` | Embedded in create/edit |

### Helpers

| Helper | File | Purpose |
|--------|------|---------|
| `productApiHelper` | `e2e/helpers/product-api.helper.ts` | API-based test data creation and cleanup |
| `productTestData` | `e2e/helpers/product-data.helper.ts` | Test data generation with faker |
| `PRODUCT_CONSTANTS` | `e2e/helpers/product-constants.helper.ts` | Known IDs, limits, enums |
| `mediaCenterHelper` | `e2e/helpers/media-center.helper.ts` | Asset upload for gallery tests |

### Spec Files

```
e2e/tests/seller-product/
  product-list/          (10 spec files, ~96 scenarios)
  product-create/        (12 spec files, ~109 scenarios)
  product-edit/          (10 spec files, ~108 scenarios)
```

Each spec stays under 200 lines, split by concern (page, validation, form, success).

### Conventions

All tests follow `e2e/conventions/` rules:
- Page objects use `readonly` locators, no assertions
- Selectors: `getByRole` > `getByLabel` > `getByText` > `getByTestId`
- Imports via `@e2e/*` path alias
- No mocking of business logic (real API calls)
- `page.route()` only for simulating network errors and specific server responses
- Arrange-Act-Assert pattern for every test
- Unique data per test for parallel safety
