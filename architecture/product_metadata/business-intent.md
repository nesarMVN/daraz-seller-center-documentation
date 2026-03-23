# Product Management — Business Intent

## What MoveOn Is

MoveOn is a cross-border commerce platform. Sellers don't manufacture products — they source from international vendors (Amazon, AliExpress, etc.) and sell to local buyers. The platform handles product discovery, purchasing, shipping, and delivery.

## The Two Product Concepts

### 1. Source Product (product-service, read-only)

The product-service is a catalog of vendor products scraped/harvested from international marketplaces. It's read-only from the seller's perspective.

- **API:** `product-api.proxy.service.moveon.global`
- **Endpoints:** search by URL, find by ID, get description — all read-only
- **Data:** title, price (in source currency), variations (from vendor), specifications, gallery, seller/vendor info, ratings
- **Purpose:** discovery — sellers browse this catalog to find products they want to sell

### 2. Seller Product (Seller module in moveon-red, new)

This is what the seller actually lists and sells. It's their product — with their pricing, their chosen variations, their images, their specifications. It may be linked to a source product, or it may be created from scratch.

- **Backend:** `moveon-red` → new `Seller` module
- **Frontend:** `moveon-global-seller` → new product management feature
- **Purpose:** the seller's own catalog that buyers see and purchase from

## Why a Separate Seller Module

The source product (product-service) is a reference. The seller product is a business entity with different concerns:

- **Pricing:** seller sets their own prices in local currency, with markup, wholesale tiers
- **Variations:** seller may not offer all vendor variations — they pick which SKUs to sell
- **Specifications:** seller may customize specs, add missing ones, or use category-defined specs from the schema
- **Images:** seller may use vendor images, upload their own (via media center), or mix both
- **Categorization:** seller assigns a shipping category (from the category tree) which determines shipping rules and attribute schema
- **Stock:** seller manages their own inventory/availability
- **Status lifecycle:** draft → pending review → active → inactive → deleted — the seller's workflow, not the vendor's

## The Category-Schema-Product Pipeline

### Step 1: Category Selection

Seller picks a shipping category from the hierarchical tree:

```
Electronics & Gadgets > Access Control System > Smart Card
```

Source: Commerce API (`/api/shipping-core/customer/shipping-category/v1/shipping-categories`)

### Step 2: Schema Lookup

The selected leaf category maps to a schema definition (`moveon-category-schemas.json`) that defines:

- **Variation attributes** — what axes create SKUs (e.g. Card Type, Color). Each has predefined options. These become the SKU matrix.
- **Specification attributes** — what product specs to collect (e.g. Brand, Frequency, Material). Each has a type (text/select), required flag, and options.

### Step 3: Dynamic Form Rendering

The frontend renders the product form based on the schema:

- Variation section: seller picks which variation values apply → generates SKU matrix → seller sets price/stock per SKU
- Specification section: form fields rendered from schema — text inputs, dropdowns, required/optional markers
- No hardcoded product forms per category — the schema drives everything

### Step 4: Product Storage

The seller product is stored in the Seller module (moveon-red) with:

- Product metadata (title, description, images, category)
- Variation properties and SKU matrix (price, stock per combination)
- Specifications (label/value pairs matching the schema)
- Link to source product ID (if sourced from catalog, nullable for manual creation)
- Ownership (agent company / seller)

## Product Creation Flows

### Flow A: Source from Catalog

```
Seller searches vendor product (by URL or keyword)
  → product-service returns source product
  → Seller picks a shipping category
  → Schema loads variation + spec attributes
  → Form pre-fills from source product data where possible
  → Seller customizes: adjusts pricing, selects variations to offer, edits specs, uploads images
  → Saves as seller product in Seller module
```

### Flow B: Create from Scratch

```
Seller clicks "New Product"
  → Picks a shipping category
  → Schema loads variation + spec attributes
  → Empty form — seller fills everything manually
  → Uploads images via media center
  → Saves as seller product in Seller module
```

## Product Management (Post-Creation)

### Status Lifecycle

| Status | Meaning |
|--------|---------|
| Draft | Incomplete, not visible to buyers |
| Pending | Submitted for review/QC |
| Active | Live, visible to buyers |
| Inactive | Taken offline by seller (stockout, seasonal, etc.) |
| Violation | Flagged for policy issues — seller must fix |
| Deleted | Permanently removed |

### Operations

- **Edit** — update any field (re-renders schema-driven form)
- **Bulk edit** — batch update pricing, stock, status
- **Search/Filter** — by name, category, status, date
- **Media management** — add/remove/reorder images (integrates with media center)

## Integration Points

### Media Center (Storage + Core modules, moveon-blue)

- Sellers upload product images/videos through the media center
- Asset picker modal lets sellers select from their media library during product creation
- Assets organized in folders, linked by folder_id

### Shipping Categories (ShippingCore module, moveon-red)

- Category tree defines product classification
- Category determines shipping rules and fees
- Category ID links to the schema for form generation

### Category Schemas (moveon-category-schemas.json)

- Defines variation_attributes and specification_attributes per leaf category
- Drives dynamic form rendering — no frontend changes needed when categories change
- Maps to Daraz and Shopify categories via metadata (future cross-platform listing)

### Product Service (product-service, separate service)

- Read-only source catalog
- Seller products may reference a source product ID
- Source data used for pre-filling during creation

### Store Module (moveon-red)

- Seller products belong to a store
- Store has region, settings, and seller identity

## What the Seller Frontend Needs

### Pages

1. **Product List** — status tabs, search/filter, bulk actions, pagination
2. **Product Create** — category picker → schema-driven form → variation/SKU builder → spec fields → image picker → save
3. **Product Edit** — same form, pre-filled with existing data

### Key Components

- **Category Picker** — hierarchical tree browser or search-as-you-type
- **Variation Builder** — select variation attributes → pick values → auto-generate SKU matrix
- **SKU Matrix Table** — per-SKU price, stock, image, availability
- **Specification Form** — dynamic fields from schema (text, select, required indicators)
- **Image Manager** — gallery with reorder (drag-drop via @dnd-kit), primary image selection, media center integration
- **Rich Text Editor** — product description
- **Status Manager** — draft/publish/deactivate controls

## Data Flow Summary

```
product-service (read-only catalog)
    ↓ source product data (optional)
Seller Module (moveon-red)
    ↑ category from ShippingCore
    ↑ schema from category-schemas
    ↑ images from Storage/Core (media center)
    ↓ seller product (owned entity)
Seller Frontend (moveon-global-seller)
    → renders dynamic forms from schema
    → manages product lifecycle
    → integrates media center for assets
```
