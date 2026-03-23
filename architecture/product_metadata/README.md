# Product Metadata Reference

Raw API responses and schemas for architecture planning. Fetched 2026-02-28.

## Files

| File | Source | Size | What |
|------|--------|------|------|
| `moveon-product-sample.json` | Product API | 17KB | Single product response (KingBank DDR5 RAM) |
| `moveon-shipping-categories.json` | Commerce API | 140KB | Full category tree (BD region, with children expanded) |
| `moveon-category-schemas.json` | Local repo | 1.7MB | Variation + specification attribute definitions per leaf category |

## API Endpoints

### Product Detail
```
GET https://product-api.proxy.service.moveon.global/api/product/customer/v1/products/find/{id}?region=BD&locale=en
```

### Shipping Categories (full tree)
```
GET https://commerce-api.proxy.service.moveon.global/api/shipping-core/customer/shipping-category/v1/shipping-categories?region=BD&expand=children&page=1&per_page=500
```

## Quick Reference: Product Structure

```
data
├── id (ULID)
├── title / title_original (bilingual)
├── slug, image, description (HTML), description_original
├── sales, stock, provider, store_id
├── shipping_category (nullable)
├── price
│   ├── original {min, max}        — range across SKUs
│   ├── discount {min, max}
│   ├── base {min, max, exchange_rate, source_currency, target_currency}
│   └── wholesales[]               — tiered pricing
├── variation
│   ├── properties[]               — axes (Color, Size, etc.)
│   │   └── values[] {id, name, color, image, thumb}
│   └── skus[]                     — purchasable units
│       ├── price {original, discount, currency, wholesales[]}
│       ├── stock {available, limit, min, availability}
│       ├── vendor_id, image, etag
│       └── property_associations[] — links SKU to property values
├── gallery[] {id, url, thumb, alt, title}
├── specifications[] {label, value}
├── ratings {count, average, frequency[]}
├── seller {id, name, link, meta (scores)}
├── country {code, name}
├── videos[], measurement, meta
└── resource_meta {created_at, updated_at}
```

Every nested object has an `object` field as type discriminator.
IDs: ULIDs for entities (product, SKU, property), UUIDs for associations/gallery.

## Quick Reference: Category Tree Structure

```
data[]
├── id (UUID)
├── name, slug
├── parent_id (null for top-level)
├── base_shipping_category_id
└── children[] (recursive, same shape)
```

~15-20 top-level categories, 3-5 levels deep.

## Quick Reference: Category Schema Structure

Keyed by category path string (e.g. `"Access Control System > Smart Card"`).

```
{
  "variation_attributes": [          — what creates SKU variants
    { name, required, options[] }
  ],
  "specification_attributes": [      — product specs
    { name, required, type (text|select), options[] }
  ],
  "reasoning": "...",                — why these attributes were chosen
  "moveon": {
    "id": "UUID",                    — maps to shipping category ID
    "name": "...",
    "path": ["Level1", "Level2", "Leaf"]
  },
  "metadata": {
    "daraz_category": null | "...",
    "shopify_category": "..."
  }
}
```

## How These Connect

1. **Category tree** defines the hierarchy sellers browse when categorizing a product
2. **Category schema** defines what variation/specification attributes a product in that category needs
3. **Product** is the result — contains the chosen variations (as properties/SKUs) and specifications (as label/value pairs)

```
Category Tree (shipping-categories)
    └── Leaf category ID
         └── Schema (category-schemas) defines required attributes
              └── Product (product-sample) has those attributes filled in
```
