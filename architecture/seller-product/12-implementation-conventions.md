# Implementation Conventions

This document defines the **mandatory** coding conventions for the Seller module. Every repository, service, enum, interface, and provider must follow these patterns exactly. These patterns are derived from established modules in the codebase (`Analytics`, `Segment`, `Core`).

---

## 1. Enums: Fields & Filters

Enums are **plain `final class`** with `public const` properties. They do **NOT** extend `Konekt\Enum\Enum`.

### Fields Enum

One per entity. Contains every column name as a constant. Used by repositories and services — never hardcode column strings.

```php
<?php

namespace MoveOn\Seller\Enum;

final class SellerProductFieldsEnum
{
    public const ID                   = 'id';
    public const TITLE                = 'title';
    public const DESCRIPTION          = 'description';
    public const STATUS               = 'status';
    public const SHIPPING_CATEGORY_ID = 'shipping_category_id';
    public const SOURCE_PRODUCT_ID    = 'source_product_id';
    public const CREATED_BY           = 'created_by';
    public const ETAG                 = 'etag';
    public const LOCK_VERSION         = 'lock_version';
    public const MIN_PRICE            = 'min_price';
    public const MAX_PRICE            = 'max_price';
    public const CREATED_AT           = 'created_at';
    public const UPDATED_AT           = 'updated_at';
}
```

### Filters Enum

One per entity. Contains only the subset of fields that are filterable.

```php
<?php

namespace MoveOn\Seller\Enum;

final class SellerProductFiltersEnum
{
    public const STATUS               = 'status';
    public const SHIPPING_CATEGORY_ID = 'shipping_category_id';
    public const TITLE                = 'title';
    public const REGION_ID            = 'region_id';
    public const OWNER_TYPE           = 'owner_type';
    public const OWNER_ID             = 'owner_id';
}
```

### Aliasing convention

When imported in repositories and services, always alias:

```php
use MoveOn\Seller\Enum\SellerProductFieldsEnum as F;
use MoveOn\Seller\Enum\SellerProductFiltersEnum as Filter;
```

---

## 2. Repository Interface

Every repository has an interface in `Repositories/Interfaces/`. It extends `BaseRepositoryInterface` and adds **typed overrides** for `create()` and `findOne()` that return the specific model type instead of the generic `Model`.

```php
<?php

namespace MoveOn\Seller\Repositories\Interfaces;

use MoveOn\Core\Repositories\Interfaces\BaseRepositoryInterface;
use MoveOn\Seller\Models\SellerProduct;

interface SellerProductRepositoryInterface extends BaseRepositoryInterface
{
    public function create(array $data): SellerProduct;

    public function findOne(array $filters, array $columns = ['*'], array $relations = []): ?SellerProduct;
}
```

Add additional method signatures here only if the repository defines custom query methods beyond what `BaseRepository` provides.

---

## 3. Repository Implementation

Every repository:
- Extends `BaseRepository`
- Implements its own `RepositoryInterface`
- Uses `FilterApplierTrait`
- Aliases Fields enum as `F`, Filters enum as `Filter`
- Returns `Builder` from `buildQuery()` (not `mixed`)
- Provides typed override wrappers for `create()` and `findOne()`

### Canonical example

```php
<?php

namespace MoveOn\Seller\Repositories;

use Illuminate\Database\Eloquent\Builder;
use MoveOn\Core\Repositories\BaseRepository;
use MoveOn\Core\Repositories\Traits\FilterApplierTrait;
use MoveOn\Seller\Models\SellerProduct;
use MoveOn\Seller\Repositories\Interfaces\SellerProductRepositoryInterface;
use MoveOn\Seller\Enum\SellerProductFieldsEnum as F;
use MoveOn\Seller\Enum\SellerProductFiltersEnum as Filter;

class SellerProductRepository extends BaseRepository implements SellerProductRepositoryInterface
{
    use FilterApplierTrait;

    protected function getModelClass(): string
    {
        return SellerProduct::class;
    }

    protected function buildQuery(array $filters = []): Builder
    {
        $query = SellerProduct::query();

        $this->applyEqualityFilters($query, $filters, [
            Filter::STATUS               => F::STATUS,
            Filter::SHIPPING_CATEGORY_ID => F::SHIPPING_CATEGORY_ID,
            Filter::OWNER_TYPE           => F::OWNER_TYPE,
            Filter::OWNER_ID             => F::OWNER_ID,
        ]);

        // Use applyRelationFilters for cross-table filters
        // Use applyListFilters for whereIn filters
        // Use applyRangeSpecs for min/max range filters
        // Use applyBooleanFilter for boolean columns

        return $query;
    }

    public function create(array $data): SellerProduct
    {
        /** @var SellerProduct $model */
        $model = parent::create($data);
        return $model;
    }

    public function findOne(array $filters, array $columns = ['*'], array $relations = []): ?SellerProduct
    {
        /** @var SellerProduct|null $model */
        $model = parent::findOne($filters, $columns, $relations);
        return $model;
    }
}
```

### FilterApplierTrait methods

The trait lives at `MoveOn\Core\Repositories\Traits\FilterApplierTrait` and provides these methods:

| Method | Purpose | Example |
|--------|---------|---------|
| `applyEqualityFilters($query, $filters, $map)` | `WHERE col = value` for each filter key | Simple column matches |
| `applyListFilters($query, $filters, $map)` | `WHERE col IN (...)` for array values | Filter by multiple IDs |
| `applyRangeSpecs($query, $filters, $specs)` | `WHERE col >= min AND col <= max` | Price range, date range |
| `applyRelationFilters($query, $filters, $map)` | Custom closure-based filters | `whereHas` for relations |
| `applyBooleanFilter($query, $filters, $key, $col)` | `WHERE col = true` when filter is truthy | `is_active` filter |

**Never use inline `->when()` chains.** Always use the trait methods.

---

## 4. Service Pattern

Services:
- Inject repository **interfaces** (not concrete classes), aliased as `Repo`
- Alias Fields enum as `Fields` or `F`
- Use `readonly` constructor promotion
- Reference columns via `Fields::COLUMN_NAME` — never hardcoded strings

### Canonical example

```php
<?php

namespace MoveOn\Seller\Services;

use MoveOn\Seller\Enum\SellerProductFieldsEnum as Fields;
use MoveOn\Seller\Repositories\Interfaces\SellerProductRepositoryInterface as Repo;
use MoveOn\Seller\Models\SellerProduct;

class SellerProductService
{
    public function __construct(
        private readonly Repo $productRepository,
        // other dependencies...
    ) {
    }

    public function getById(int $id, string $ownerType, int $ownerId): ?SellerProduct
    {
        return $this->productRepository->findOne([
            Fields::ID    => $id,
            'owner_type'  => $ownerType,
            'owner_id'    => $ownerId,
        ]);
    }
}
```

### Key rules for services

1. **Never inject concrete repository classes** — always inject the interface
2. **Use enum constants for all field references** — `Fields::STATUS`, not `'status'`
3. **Use `readonly` on all constructor-promoted properties**
4. **Repository method calls use the BaseRepository API** — `findOne()`, `findAll()`, `create()`, `update()`, `delete()`, `updateMany()`, `deleteMany()`, `count()`, `exists()`, `findByIds()`, `insertMany()`, `findAllWithoutPagination()`

---

## 5. Service Provider Bindings

The module's `SellerServiceProvider` must register interface-to-implementation bindings and singleton services.

```php
<?php

namespace MoveOn\Seller\Providers;

use Illuminate\Support\ServiceProvider;

class SellerServiceProvider extends ServiceProvider
{
    public function boot()
    {
        $this->loadMigrationsFrom(__DIR__ . '/../Database/Migrations');
        $this->loadRoutesFrom(__DIR__ . '/../Http/agent-routes.php');
    }

    public function register()
    {
        $this->registerBindings();
        $this->registerServices();
    }

    protected function registerBindings()
    {
        // Bind repository interfaces to implementations
        $this->app->bind(
            \MoveOn\Seller\Repositories\Interfaces\SellerProductRepositoryInterface::class,
            \MoveOn\Seller\Repositories\SellerProductRepository::class
        );
        // ... one bind() per repository
    }

    protected function registerServices()
    {
        // Register services as singletons (optional — only if needed)
        // Services with only interface-bound dependencies can often
        // be resolved automatically by the container without explicit registration.
    }
}
```

---

## 6. Directory Structure

```
modules/MoveOn/Seller/src/
├── Enum/
│   ├── SellerProductFieldsEnum.php          # final class, public const
│   ├── SellerProductFiltersEnum.php         # final class, public const
│   ├── SellerProductSkuFieldsEnum.php       # final class, public const
│   ├── SellerProductSkuFiltersEnum.php      # final class, public const
│   └── ...                                  # one Fields + one Filters per entity
├── Repositories/
│   ├── Interfaces/
│   │   ├── SellerProductRepositoryInterface.php
│   │   ├── SellerProductSkuRepositoryInterface.php
│   │   └── ...                              # one interface per repository
│   ├── SellerProductRepository.php
│   ├── SellerProductSkuRepository.php
│   └── ...
├── Services/
│   ├── SellerProductService.php
│   ├── SellerProductSkuService.php
│   ├── SellerProductEtagService.php
│   ├── CategorySchemaValidator.php
│   ├── JsonPatchProcessor.php
│   └── ...
└── Providers/
    └── SellerServiceProvider.php             # registerBindings() + registerServices()
```

---

## 7. BaseRepository API Reference

These are the methods available on `BaseRepository` (from `MoveOn\Core\Repositories\BaseRepository`). Services should use these via the repository interface — **never bypass the repository to query models directly**.

| Method | Signature | Description |
|--------|-----------|-------------|
| `findOne` | `findOne(array $filters, array $columns = ['*'], array $relations = []): ?Model` | Find single record |
| `findAll` | `findAll(int $page, int $perPage, array $filters = [], array $columns = ['*'], array $relations = [], string $sortField = 'created_at', string $sortDirection = 'asc'): LengthAwarePaginator` | Paginated list |
| `findAllWithoutPagination` | `findAllWithoutPagination(array $filters = [], array $columns = ['*'], array $relations = [], ?string $sortField = 'created_at', ?string $sortDirection = 'asc'): Collection` | Unpaginated list |
| `create` | `create(array $data): Model` | Insert one record |
| `update` | `update(int $id, array $data): Model` | Update by ID |
| `delete` | `delete(int $id): bool` | Delete by ID |
| `insertMany` | `insertMany(array $records): bool` | Bulk insert |
| `updateMany` | `updateMany(array $filters, array $data): int` | Bulk update by filter |
| `deleteMany` | `deleteMany(array $filters): int` | Bulk delete by filter |
| `exists` | `exists(array $filters = []): bool` | Check existence |
| `count` | `count(array $filters = []): int` | Count records |
| `findByIds` | `findByIds(array $ids, array $columns = ['*'], array $relations = []): Collection` | Find by ID array |

---

## 8. Commenting & Documentation

**Use docblocks on methods. Do NOT scatter inline comments line-by-line through method bodies.**

Code should be self-documenting through clear naming. If a method needs explanation, put it in the docblock — not as inline `//` comments narrating each step.

### Good — docblock describes the method's purpose

```php
/**
 * Create a new seller product with all nested resources in a single transaction.
 */
public function create(array $payload, string $ownerType, int $ownerId, int $createdBy): SellerProduct
{
    DB::beginTransaction();

    try {
        $product = $this->productRepository->create([...]);

        $lookupMap = [];
        if (!empty($payload['properties'])) {
            $lookupMap = $this->propertyService->syncProperties($product, $payload['properties']);
        }

        $skuIdMap = [];
        if (!empty($payload['skus'])) {
            $skuIdMap = $this->skuService->syncSkus($product, $payload['skus'], $lookupMap);
        }

        $this->etagService->computeAndStore($product);

        DB::commit();
    } catch (Exception $exception) {
        DB::rollBack();
        throw new SellerProductCreateException($exception->getMessage());
    }

    return $product->load([...]);
}
```

### Bad — inline comments narrating every step

```php
public function create(array $payload, string $ownerType, int $ownerId, int $createdBy): SellerProduct
{
    DB::beginTransaction();

    try {
        // Create the product
        $product = $this->productRepository->create([...]);

        // Sync properties and build lookup map
        $lookupMap = [];
        if (!empty($payload['properties'])) {
            $lookupMap = $this->propertyService->syncProperties($product, $payload['properties']);
        }

        // Sync SKUs using the lookup map and get ID map for regional prices
        $skuIdMap = [];
        if (!empty($payload['skus'])) {
            $skuIdMap = $this->skuService->syncSkus($product, $payload['skus'], $lookupMap);
        }

        // Compute and store ETag
        $this->etagService->computeAndStore($product);

        DB::commit();
    } catch (Exception $exception) {
        DB::rollBack();
        throw new SellerProductCreateException($exception->getMessage());
    }

    // Reload with all relations
    return $product->load([...]);
}
```

### Rules

1. **Every public method** must have a docblock (`/** ... */`) describing its purpose
2. **No inline `//` comments** that simply restate what the next line does — the code speaks for itself
3. **Inline comments are only acceptable** when explaining a non-obvious *why* — e.g. a workaround, a business rule that isn't obvious from the code, or a framework quirk
4. **Do not add `@param`/`@return` tags** when the types are already declared in the method signature — PHP type hints are the source of truth

---

## 9. Anti-patterns (DO NOT do these)

| Anti-pattern | Correct approach |
|-------------|-----------------|
| `extends Konekt\Enum\Enum` for Fields/Filters | `final class` with `public const` |
| `->when(isset(...))` chains in `buildQuery()` | `FilterApplierTrait` methods |
| Injecting concrete `SellerProductRepository` in services | Inject `SellerProductRepositoryInterface` |
| Returning `mixed` from `buildQuery()` | Return `Builder` |
| Hardcoding `'status'` string in service methods | Use `Fields::STATUS` |
| No repository interface | Always create an interface extending `BaseRepositoryInterface` |
| No `registerBindings()` in service provider | Add interface→implementation bindings |
| Custom query methods that duplicate `BaseRepository` API | Use inherited methods (`findOne`, `updateMany`, etc.) |
| Line-by-line `//` inline comments narrating code | Docblock on the method; let code be self-documenting |
| `@param`/`@return` tags duplicating PHP type hints | Rely on method signature type declarations |

---

## 10. Reference Files

These are the canonical reference implementations in the codebase. When in doubt, study these:

| What | File |
|------|------|
| Repository implementation | `modules/MoveOn/Analytics/src/Repositories/MetabaseSyncLogRepository.php` |
| Repository interface | `modules/MoveOn/Analytics/src/Repositories/Interfaces/MetabaseSyncLogRepositoryInterface.php` |
| FilterApplierTrait | `modules/MoveOn/Core/src/Repositories/Traits/FilterApplierTrait.php` |
| BaseRepository | `modules/MoveOn/Core/src/Repositories/BaseRepository.php` |
| BaseRepositoryInterface | `modules/MoveOn/Core/src/Repositories/Interfaces/BaseRepositoryInterface.php` |
| Service pattern | `modules/MoveOn/Segment/src/Services/SegmentRefreshPartitionService.php` |
| Service provider bindings | `modules/MoveOn/Analytics/src/Providers/AnalyticsServiceProvider.php` |
| Fields enum | `modules/MoveOn/Analytics/src/Enum/MetabaseSyncLogFields.php` |
| Filters enum | `modules/MoveOn/Analytics/src/Enum/MetabaseSyncLogFilters.php` |
