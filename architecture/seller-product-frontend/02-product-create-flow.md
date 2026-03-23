# Product Create Flow

## Overview

Product creation is a multi-section form on a single page. The seller fills in sections progressively: select category, enter basic info, define variations (properties + SKUs), add specifications, manage gallery images, and set regional pricing. The form can be saved as `draft` at any time or submitted as `pending` when all required fields are filled.

**Route:** `/[locale]/(seller)/products/create`
**Route (sourced):** `/[locale]/(seller)/products/create?source_product_id={ulid}`

---

## Two Creation Flows

### Flow A: Sourced from Vendor Catalog

```
┌──────────────────────┐     ┌──────────────────────┐
│ Seller searches      │     │ product-service API   │
│ vendor product by    │────>│ returns source data   │
│ URL or keyword       │     │ (title, specs, imgs)  │
└──────────┬───────────┘     └──────────┬────────────┘
           │                            │
           │         ┌─────────────────────────────┐
           └────────>│ Navigate to:                 │
                     │ /products/create             │
                     │   ?source_product_id={ulid}  │
                     └──────────┬──────────────────┘
                                │
                                ▼
                     ┌──────────────────────┐
                     │ Fetch source product  │
                     │ Pre-fill form fields  │
                     │ User reviews + edits  │
                     └──────────────────────┘
```

### Flow B: Manual Creation

```
┌──────────────────────┐
│ Seller clicks         │
│ [+ New Product]       │
│ from product list     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Navigate to:          │
│ /products/create      │
│ (no source param)     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Empty form            │
│ Start with category   │
│ selection             │
└──────────────────────┘
```

---

## Full Page Layout

```
╔═════════════════════════════════════════════════════════════════════════════╗
║  MoveOn Seller Center                              John Doe   [Bell] [Cog]  ║
╠═════════════════════════════════════════════════════════════════════════════╣
║                                                                             ║
║  Products > Create Product                                                  ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  SECTION 1: CATEGORY                                                │   ║
║  │                                                                     │   ║
║  │  Shipping Category *                                                │   ║
║  │  ┌───────────────────────────────────────────────────────────────┐  │   ║
║  │  │ Electronics & Gadgets > Computer Peripherals > Mouse     [x] │  │   ║
║  │  └───────────────────────────────────────────────────────────────┘  │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  SECTION 2: BASIC INFORMATION                                       │   ║
║  │                                                                     │   ║
║  │  Product Title *                                                    │   ║
║  │  ┌───────────────────────────────────────────────────────────┐     │   ║
║  │  │ Wireless Gaming Mouse 2.4GHz RGB Ergonomic_______________│     │   ║
║  │  └───────────────────────────────────────────────────────────┘     │   ║
║  │  48/255 characters                                                  │   ║
║  │                                                                     │   ║
║  │  Description                                                        │   ║
║  │  ┌───────────────────────────────────────────────────────────┐     │   ║
║  │  │ Rich text editor                                          │     │   ║
║  │  │ [B] [I] [U] [H1] [H2] [Ul] [Ol] [Link] [Img]            │     │   ║
║  │  │                                                           │     │   ║
║  │  │ Professional wireless gaming mouse with RGB lighting...   │     │   ║
║  │  │                                                           │     │   ║
║  │  └───────────────────────────────────────────────────────────┘     │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  SECTION 3: VARIATIONS & SKUs                                       │   ║
║  │  (See 04-variation-sku-builder.md for full detail)                  │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  SECTION 4: SPECIFICATIONS                                          │   ║
║  │  (Category-driven fields -- see inline detail below)                │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  SECTION 5: GALLERY                                                 │   ║
║  │  (See 05-gallery-manager.md for full detail)                        │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  SECTION 6: REGIONAL PRICING                                        │   ║
║  │  (See 06-regional-pricing.md for full detail)                       │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │  ACTION BAR (sticky bottom)                                         │   ║
║  │                                                                     │   ║
║  │                          [Save as Draft]  [Submit for Review]       │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
║                                                                             ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### Form Completion Sidebar

```
╔════════════════════════════════════════════════════════════════════════╗
║  FORM COMPLETION TRACKER (right sidebar panel)                        ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  In the full page layout, a sticky sidebar panel tracks section        ║
║  completion status:                                                    ║
║                                                                        ║
║  ┌──────────────────────────────┐                                     ║
║  │  Product Completeness        │                                     ║
║  │                              │                                     ║
║  │  ████████░░░░░░░░  50%       │  ← progress bar                    ║
║  │  3 of 6 sections complete    │                                     ║
║  │                              │                                     ║
║  │  ● Category         ✓       │  ← filled circle = complete          ║
║  │  ◐ Basic Info       !       │  ← half circle = partial             ║
║  │  ● Variations       ✓       │                                     ║
║  │  ○ Specifications           │  ← empty circle = not started        ║
║  │  ○ Gallery                  │                                     ║
║  │  ○ Regional Pricing         │                                     ║
║  │                              │                                     ║
║  │  ──────────────────────────  │                                     ║
║  │  Quality Score: Good         │                                     ║
║  │  ████████████░░░░  75%       │                                     ║
║  │                              │                                     ║
║  │  Missing for "Excellent":    │                                     ║
║  │  · Add 3+ gallery images     │                                     ║
║  │  · Fill optional specs       │                                     ║
║  │  · Add regional pricing      │                                     ║
║  └──────────────────────────────┘                                     ║
║                                                                        ║
║  Section status logic:                                                 ║
║  ● Complete: all required fields filled, no validation errors          ║
║  ◐ Partial: some fields filled but incomplete                          ║
║  ○ Empty: no data entered                                              ║
║                                                                        ║
║  Quality score tiers:                                                  ║
║  - Poor (0-33%): only required fields, no extras                       ║
║  - Good (34-74%): required fields + some optional                      ║
║  - Excellent (75-100%): rich description, gallery, specs, pricing      ║
║                                                                        ║
║  The sidebar scrolls with the page and highlights the current          ║
║  section. Clicking a section name scrolls to that form section.        ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Section 1: Category Picker

### Recently Used Categories

```
╔════════════════════════════════════════════════════════════════════════╗
║  RECENTLY USED CATEGORIES                                             ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Before opening the full category tree, show recently used categories  ║
║  for quick selection (above the [Browse] button):                      ║
║                                                                        ║
║  Shipping Category *                                                   ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ Select a category...                              [Browse]│        ║
║  └───────────────────────────────────────────────────────────┘        ║
║                                                                        ║
║  Recently used:                                                        ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │  Electronics > Computer Peripherals > Mouse              [Select] │ ║
║  │  Apparel > Men's Clothing > T-Shirts                     [Select] │ ║
║  │  Health & Beauty > Skincare > Face Cream                 [Select] │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║                                                                        ║
║  - Shows last 5 categories the seller selected (most recent first).   ║
║  - Stored in localStorage key: "recent-categories".                   ║
║  - Value: JSON array of { id, path_label, selected_at } objects.      ║
║  - One-click [Select] bypasses the category tree modal entirely.      ║
║  - Panel hidden if no recent categories exist (first-time user).      ║
║  - Updated on every successful category selection (tree or recent).   ║
║  - Duplicate entries deduplicated by category id; oldest removed       ║
║    when list exceeds 5.                                                ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

> Future: Shipping/warranty/return policy per product — not in MVP.

### Workflow

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: Empty category field                                         ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Shipping Category *                                                   ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ Select a category...                              [Browse]│        ║
║  └───────────────────────────────────────────────────────────┘        ║
║                                                                        ║
║  Category is required before other sections become editable.           ║
║  All sections below are disabled/collapsed until category is chosen.   ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝

### Category Picker Interaction States

```
╔════════════════════════════════════════════════════════════════════════╗
║  CATEGORY PICKER ELEMENT STATES                                       ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  [Browse] button:                                                      ║
║                                                                        ║
║  DEFAULT              HOVER                POPULATED                   ║
║  ┌──────────┐        ┌──────────┐         ┌────────────────────────┐  ║
║  │  Browse  │        │  Browse  │         │ Electronics > Mouse [x]│  ║
║  │(outline) │        │(blue bg) │         │ (filled, [x] to clear) │  ║
║  └──────────┘        └──────────┘         └────────────────────────┘  ║
║                                                                        ║
║  Tree items in modal:                                                  ║
║                                                                        ║
║  PARENT (collapsed)   PARENT (expanded)                                ║
║  ▶ Electronics        ▼ Electronics                                    ║
║  (cursor pointer,     (chevron rotates,                                ║
║   bold text)           children visible)                               ║
║                                                                        ║
║  LEAF (default)       LEAF (hover)        LEAF (selected)              ║
║  · Mouse              · Mouse             · Mouse                      ║
║  (normal weight,      (blue-50 bg,        (blue-100 bg, blue text,     ║
║   cursor pointer)      underline)          ✓ check icon left)         ║
║                                                                        ║
║  NON-LEAF (not selectable)                                             ║
║  ▶ Computer Peripherals                                                ║
║  (clicking expands/collapses only, cannot be selected as category)     ║
║                                                                        ║
║  Search input in modal:                                                ║
║  DEFAULT: "Search categories..." placeholder                           ║
║  FOCUSED: blue border, cursor active                                   ║
║  TYPING: filters tree in real-time, non-matching nodes hidden          ║
║  NO RESULTS: "No categories match your search" message                 ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

                                    │
                          User clicks [Browse]
                                    │
                                    ▼
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 2: Category picker modal opens                                  ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ╔══════════════════════════════════════════════════════════════╗     ║
║  ║  Select Shipping Category                          [x]      ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                                              ║     ║
║  ║  ┌────────────────────────────────────────────────────────┐ ║     ║
║  ║  │ Search categories...                                    │ ║     ║
║  ║  └────────────────────────────────────────────────────────┘ ║     ║
║  ║                                                              ║     ║
║  ║  ┌────────────────────────────────────────────────────────┐ ║     ║
║  ║  │                                                        │ ║     ║
║  ║  │  v Electronics & Gadgets                               │ ║     ║
║  ║  │    v Computer Peripherals                              │ ║     ║
║  ║  │      - Keyboard                                        │ ║     ║
║  ║  │      - Mouse        <-- leaf node (selectable)         │ ║     ║
║  ║  │      - Headset                                         │ ║     ║
║  ║  │      - Webcam                                          │ ║     ║
║  ║  │    v Smart Home                                        │ ║     ║
║  ║  │      - Smart Plug                                      │ ║     ║
║  ║  │      - Smart Light                                     │ ║     ║
║  ║  │  v Apparel & Accessories                               │ ║     ║
║  ║  │    v Men's Clothing                                    │ ║     ║
║  ║  │      - T-Shirts                                        │ ║     ║
║  ║  │      - Jeans                                           │ ║     ║
║  ║  │  v Health & Beauty                                     │ ║     ║
║  ║  │    ...                                                 │ ║     ║
║  ║  │                                                        │ ║     ║
║  ║  └────────────────────────────────────────────────────────┘ ║     ║
║  ║                                                              ║     ║
║  ║  Only leaf nodes (no children) are selectable.               ║     ║
║  ║  Parent nodes expand/collapse on click.                      ║     ║
║  ║                                                              ║     ║
║  ║  Selected: Electronics & Gadgets > Computer Peripherals >    ║     ║
║  ║           Mouse                                              ║     ║
║  ║                                                              ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                          [Cancel]  [Select Category]        ║     ║
║  ╚══════════════════════════════════════════════════════════════╝     ║
║                                                                        ║
║  API Data Source:                                                      ║
║  GET /api/shipping-core/customer/shipping-category/v1/                 ║
║      shipping-categories                                               ║
║                                                                        ║
║  Returns 18 top-level categories with nested children.                 ║
║  Cached with TanStack Query (staleTime: 30 minutes).                   ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
                                    │
                        User clicks [Select Category]
                                    │
                                    ▼
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 3: Category selected, form sections unlock                      ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Shipping Category *                                                   ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ Electronics & Gadgets > Computer Peripherals > Mouse  [x] │        ║
║  └───────────────────────────────────────────────────────────┘        ║
║                                                                        ║
║  What happens after category selection:                                ║
║  1. Load category schema from moveon-category-schemas.json             ║
║     The JSON file is keyed by category PATH string                     ║
║     (e.g., "Access Control System > Smart Card"), not by UUID.         ║
║     Each schema entry has a moveon.id field (the UUID).                ║
║     Build a UUID-to-schema map at load time:                           ║
║       Object.fromEntries(                                              ║
║         Object.values(schemas).map(s => [s.moveon.id, s])             ║
║       )                                                                ║
║     Then look up by shipping_category_id.                              ║
║     The schema JSON can be bundled client-side (static asset)          ║
║     or served via API. It changes infrequently.                        ║
║  2. Populate variation property dropdowns from                         ║
║     schema.variation_attributes                                        ║
║  3. Populate specification fields from                                 ║
║     schema.specification_attributes                                    ║
║  4. Enable all form sections below                                     ║
║                                                                        ║
║  If category has a schema:                                             ║
║  - Property names are pre-defined (e.g., "Color", "DPI")              ║
║  - Property value options are constrained to schema options            ║
║  - Required properties (schema required:true) MUST be present          ║
║    when submitting as "pending" (enforced by server)                   ║
║  - Required specs are marked with asterisk                             ║
║  - Required specs MUST have values when submitting as "pending"        ║
║  - Select-type specs show dropdown with schema options                 ║
║                                                                        ║
║  If category has NO schema:                                            ║
║  - Property names are free-text                                        ║
║  - Property values are free-text                                       ║
║  - Specifications are free-text                                        ║
║  - No required properties or specs enforced by schema                  ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Category Change Warning

```
╔════════════════════════════════════════════════════════════════════════╗
║  User clicks [x] on existing category to change it                    ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ╔══════════════════════════════════════════════════════════════╗     ║
║  ║  Change Category?                                   [x]     ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                                              ║     ║
║  ║  Changing the category will:                                 ║     ║
║  ║                                                              ║     ║
║  ║  - Reset all variation properties and values                 ║     ║
║  ║  - Remove all SKU associations and SKUs                      ║     ║
║  ║  - Reset all regional pricing (tied to SKUs)                 ║     ║
║  ║  - Clear category-specific specification fields              ║     ║
║  ║                                                              ║     ║
║  ║  Basic info (title, description) and gallery                 ║     ║
║  ║  will be preserved.                                          ║     ║
║  ║                                                              ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                  [Cancel]  [Change Category]                ║     ║
║  ╚══════════════════════════════════════════════════════════════╝     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Section 2: Basic Information

```
╔════════════════════════════════════════════════════════════════════════╗
║  BASIC INFORMATION                                                     ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Product Title *                                                       ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ Wireless Gaming Mouse 2.4GHz RGB Ergonomic_______________│        ║
║  └───────────────────────────────────────────────────────────┘        ║
║  48/255 characters                                                     ║
║                                                                        ║
║  Validation:                                                           ║
║  - Required                                                            ║
║  - 1-255 characters                                                    ║
║  - Client-side: show character counter, disable submit > 255           ║
║                                                                        ║
║  > Future: Multi-language product content (title, description          ║
║  > per locale) — not in MVP.                                           ║
║                                                                        ║
║  Description                                                           ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ ┌─────────────────────────────────────────────────────┐   │        ║
║  │ │ [B] [I] [U] [H1] [H2] [Ul] [Ol] [Link] [Img]      │   │        ║
║  │ └─────────────────────────────────────────────────────┘   │        ║
║  │                                                           │        ║
║  │ <p>Professional wireless gaming mouse with <b>RGB</b>     │        ║
║  │ lighting and ergonomic design...</p>                      │        ║
║  │                                                           │        ║
║  │                                                           │        ║
║  └───────────────────────────────────────────────────────────┘        ║
║  Rich HTML editor. Max 65,535 characters. Optional.                    ║
║                                                                        ║
║  > Future: Product highlights/key features bullet list — not in MVP.   ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Form Field Validation States

```
╔════════════════════════════════════════════════════════════════════════╗
║  TITLE INPUT — 6 INTERACTION STATES                                   ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  1. DEFAULT                                                            ║
║  Product Title *                                                       ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ Enter product title...                                     │        ║
║  └───────────────────────────────────────────────────────────┘        ║
║  (gray border #d1d5db, placeholder text gray #9ca3af)                 ║
║                                                                        ║
║  2. FOCUSED                                                            ║
║  Product Title *                                                       ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ |                                                (cursor)  │        ║
║  └───────────────────────────────────────────────────────────┘        ║
║  0/255 characters                                                      ║
║  (blue border #3b82f6, 2px ring, char counter appears on focus)        ║
║                                                                        ║
║  3. VALID                                                              ║
║  Product Title *                                                       ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ Wireless Gaming Mouse 2.4GHz RGB Ergonomic            ✓   │        ║
║  └───────────────────────────────────────────────────────────┘        ║
║  48/255 characters                                                     ║
║  (green border #22c55e, ✓ check icon right-aligned)                   ║
║                                                                        ║
║  4. ERROR — REQUIRED                                                   ║
║  Product Title *                                                       ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │                                                        ⚠️  │        ║
║  └───────────────────────────────────────────────────────────┘        ║
║  ⚠️ The title field is required.                                      ║
║  (red border #ef4444, ⚠️ icon, red error text below)                  ║
║                                                                        ║
║  5. ERROR — TOO LONG                                                   ║
║  Product Title *                                                       ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ Very long product title that exceeds the maximum char...⚠️ │        ║
║  └───────────────────────────────────────────────────────────┘        ║
║  260/255 characters  ← red counter when over limit                     ║
║  ⚠️ Title must be 255 characters or fewer.                            ║
║  (red border, counter turns red and bold at 256+)                      ║
║                                                                        ║
║  6. DISABLED                                                           ║
║  Product Title *                                                       ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│        ║
║  └───────────────────────────────────────────────────────────┘        ║
║  Select a category first                                               ║
║  (gray bg #f3f4f6, cursor not-allowed, hint text below)                ║
║                                                                        ║
║  Character counter behavior:                                           ║
║  - Hidden by default, appears on focus                                 ║
║  - Gray text below 200 chars                                           ║
║  - Orange text at 230-254 chars (approaching limit)                    ║
║  - Red bold text at 255+ chars (over limit)                            ║
║  - Format: "48/255 characters"                                         ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Rich Text Editor States

```
╔════════════════════════════════════════════════════════════════════════╗
║  DESCRIPTION EDITOR — INTERACTION STATES                              ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  EMPTY (placeholder visible):                                          ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ ┌─────────────────────────────────────────────────────┐   │        ║
║  │ │ [B] [I] [U] [H1] [H2] [Ul] [Ol] [Link] [Img]      │   │        ║
║  │ └─────────────────────────────────────────────────────┘   │        ║
║  │                                                           │        ║
║  │  Enter product description...                             │        ║
║  │  (italic, gray #9ca3af placeholder)                       │        ║
║  │                                                           │        ║
║  └───────────────────────────────────────────────────────────┘        ║
║  (gray border #d1d5db)                                                 ║
║                                                                        ║
║  FOCUSED (blue ring, cursor active):                                   ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ ┌─────────────────────────────────────────────────────┐   │        ║
║  │ │ [B] [I] [U] [H1] [H2] [Ul] [Ol] [Link] [Img]      │   │        ║
║  │ └─────────────────────────────────────────────────────┘   │        ║
║  │                                                           │        ║
║  │  |  (blinking cursor)                                     │        ║
║  │                                                           │        ║
║  └───────────────────────────────────────────────────────────┘        ║
║  (blue border #3b82f6, 2px ring)                                       ║
║                                                                        ║
║  WITH CONTENT (toolbar active states):                                 ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ ┌─────────────────────────────────────────────────────┐   │        ║
║  │ │ [B] [I] [U] [H1] [H2] [Ul] [Ol] [Link] [Img]      │   │        ║
║  │ │  ^         ^                                         │   │        ║
║  │ │  active    active (blue bg when formatting applied)  │   │        ║
║  │ └─────────────────────────────────────────────────────┘   │        ║
║  │                                                           │        ║
║  │  Professional wireless gaming mouse with RGB lighting     │        ║
║  │  and ergonomic design for extended gaming sessions.       │        ║
║  │                                                           │        ║
║  └───────────────────────────────────────────────────────────┘        ║
║                                                                        ║
║  Toolbar button states:                                                ║
║  DEFAULT: gray icon, transparent bg                                    ║
║  HOVER: darker icon, light gray bg                                     ║
║  ACTIVE (formatting applied): blue icon, blue-50 bg                    ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Rich Text Paste Handling

```
╔════════════════════════════════════════════════════════════════════════╗
║  PASTE BEHAVIOR IN DESCRIPTION EDITOR                                 ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Plain text paste (Ctrl+Shift+V or plain clipboard):                   ║
║  - Strips all formatting, inserts as plain text.                       ║
║                                                                        ║
║  Rich text paste (Ctrl+V from a web page or another editor):           ║
║  - Preserves: bold, italic, underline, headings, lists (ul/ol), links. ║
║  - Strips: unsafe HTML tags (script, iframe, style, event handlers).   ║
║  - Sanitization via DOMPurify or equivalent library.                   ║
║                                                                        ║
║  Paste from Word / Google Sheets:                                      ║
║  - Sanitizes proprietary tags (mso-*, google-sheets-*).               ║
║  - Preserves basic structure (paragraphs, bold, lists).                ║
║  - Removes inline styles (font-family, color, font-size).             ║
║                                                                        ║
║  Max length enforcement on paste:                                      ║
║  - If pasted content would exceed 65,535 characters, it is truncated.  ║
║  - Toast warning: "Content was trimmed to fit the 65,535 char limit."  ║
║  - The character counter updates immediately to show the truncated     ║
║    length.                                                             ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Inline Help and Field Hints

```
╔════════════════════════════════════════════════════════════════════════╗
║  INLINE HELP PATTERNS                                                 ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Pattern 1: Tooltip icon (ⓘ)                                         ║
║                                                                        ║
║  Product Title * ⓘ                                                    ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ Wireless Gaming Mouse...                                   │        ║
║  └───────────────────────────────────────────────────────────┘        ║
║                                                                        ║
║  On hover over ⓘ:                                                     ║
║  ┌──────────────────────────────────────┐                             ║
║  │ Use a clear, descriptive title.      │ ← popover (dark bg,         ║
║  │ Include key features, brand name,    │   white text, arrow          ║
║  │ and product type. This appears       │   pointing to ⓘ icon)       ║
║  │ in search results and listings.      │                              ║
║  └──────────────────────────────────────┘                             ║
║                                                                        ║
║  Pattern 2: Helper text below input                                    ║
║                                                                        ║
║  Product Title *                                                       ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ Wireless Gaming Mouse...                                   │        ║
║  └───────────────────────────────────────────────────────────┘        ║
║  48/255 characters                                                     ║
║  Include brand, product type, and key features for better search.      ║
║  (gray text, smaller font, always visible)                             ║
║                                                                        ║
║  Pattern 3: Expandable example                                         ║
║                                                                        ║
║  Product Title *                                                       ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ ___________________________________________________________│        ║
║  └───────────────────────────────────────────────────────────┘        ║
║  [See Example ▼]                                                       ║
║                                                                        ║
║  On click [See Example ▼] expands to [See Example ▲]:                  ║
║  ┌──────────────────────────────────────────────────────────┐         ║
║  │  Good: "Samsung Galaxy S24 Ultra 256GB Titanium Black"    │         ║
║  │  Bad:  "phone"                                            │         ║
║  │  Bad:  "BEST DEAL!!! Amazing phone you MUST BUY!!!"       │         ║
║  └──────────────────────────────────────────────────────────┘         ║
║                                                                        ║
║  Pattern 4: Range hint in field                                        ║
║                                                                        ║
║  Price *                                                               ║
║  ┌───────────────────────────────────────────────────────────┐        ║
║  │ 0.00                                                       │        ║
║  └───────────────────────────────────────────────────────────┘        ║
║  $0.01 – $9,999,999,999.99                                             ║
║  (gray range hint below, always visible for numeric fields)            ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Section 3: Variations & SKUs

Covered in detail in [04-variation-sku-builder.md](./04-variation-sku-builder.md).

Quick summary of what appears in this section:

```
┌─────────────────────────────────────────────────────────────────────┐
│  VARIATIONS & SKUs                                                  │
│                                                                     │
│  Properties:                                                        │
│  ┌──────────────────────────────────────────┐                       │
│  │ Color *       [Black] [White] [Red] [+]  │  Drag to reorder     │
│  │ DPI           [800] [1600] [3200] [+]    │  values               │
│  │ [+ Add Property]                         │                       │
│  └──────────────────────────────────────────┘                       │
│                                                                     │
│  SKU Matrix (6 combinations):                                       │
│  ┌─────┬───────┬──────┬───────┬───────┬──────────┬────────┐        │
│  │     │ Color │ DPI  │ Price │ Stock │ SKU Code │ Active │        │
│  ├─────┼───────┼──────┼───────┼───────┼──────────┼────────┤        │
│  │  =  │ Black │ 800  │ 24.99 │ 100   │ BK-800   │ [on]   │        │
│  │  =  │ Black │ 1600 │ 29.99 │ 80    │ BK-1600  │ [on]   │        │
│  │  =  │ Black │ 3200 │ 39.99 │ 50    │ BK-3200  │ [on]   │        │
│  │  =  │ White │ 800  │ 24.99 │ 100   │ WH-800   │ [on]   │        │
│  │  =  │ White │ 1600 │ 29.99 │ 80    │ WH-1600  │ [on]   │        │
│  │  =  │ White │ 3200 │ 39.99 │ 50    │ WH-3200  │ [on]   │        │
│  └─────┴───────┴──────┴───────┴───────┴──────────┴────────┘        │
│  = drag handle for reorder                                          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Section 4: Specifications (Inline)

Specifications are category-driven. When a category with a schema is selected, the form shows fields defined in `specification_attributes`.

```
╔════════════════════════════════════════════════════════════════════════╗
║  SPECIFICATIONS                                                        ║
║  (Fields from category schema for "Mouse")                             ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌─────────────────────────────────────────────────────────────┐      ║
║  │  Brand *              (type: text, required)                 │      ║
║  │  ┌─────────────────────────────────────────────────────┐    │      ║
║  │  │ Logitech____________________________________________│    │      ║
║  │  └─────────────────────────────────────────────────────┘    │      ║
║  ├─────────────────────────────────────────────────────────────┤      ║
║  │  Connectivity *       (type: select, required)               │      ║
║  │  ┌─────────────────────────────────────────────────────┐    │      ║
║  │  │ Wireless (2.4GHz)                              [v]  │    │      ║
║  │  └─────────────────────────────────────────────────────┘    │      ║
║  │  Options: Wired, Wireless (2.4GHz), Bluetooth, Dual Mode    │      ║
║  ├─────────────────────────────────────────────────────────────┤      ║
║  │  Sensor Type          (type: select, optional)               │      ║
║  │  ┌─────────────────────────────────────────────────────┐    │      ║
║  │  │ Optical                                        [v]  │    │      ║
║  │  └─────────────────────────────────────────────────────┘    │      ║
║  │  Options: Optical, Laser, Other                              │      ║
║  ├─────────────────────────────────────────────────────────────┤      ║
║  │  DPI Range            (type: text, optional)                 │      ║
║  │  ┌─────────────────────────────────────────────────────┐    │      ║
║  │  │ 800-3200____________________________________________│    │      ║
║  │  └─────────────────────────────────────────────────────┘    │      ║
║  ├─────────────────────────────────────────────────────────────┤      ║
║  │  Weight               (type: text, optional)                 │      ║
║  │  ┌─────────────────────────────────────────────────────┐    │      ║
║  │  │ 85g_________________________________________________│    │      ║
║  │  └─────────────────────────────────────────────────────┘    │      ║
║  ├─────────────────────────────────────────────────────────────┤      ║
║  │  Warranty             (type: text, optional)                 │      ║
║  │  ┌─────────────────────────────────────────────────────┐    │      ║
║  │  │ 2 Years_____________________________________________│    │      ║
║  │  └─────────────────────────────────────────────────────┘    │      ║
║  └─────────────────────────────────────────────────────────────┘      ║
║                                                                        ║
║  [+ Add Custom Specification]                                          ║
║  (allows adding specs beyond what the schema defines)                  ║
║                                                                        ║
║  Drag handle (=) on left of each spec row to reorder positions.        ║
║                                                                        ║
║  Validation:                                                           ║
║  - Required fields: name + value must be non-empty                     ║
║  - Name: max 100 characters                                            ║
║  - Value: max 500 characters                                           ║
║  - Select fields: value must be one of schema options                  ║
║  - Text fields: any non-empty string                                   ║
║  - Max 50 specifications total                                         ║
║  - No duplicate names                                                  ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

> Future: Specification fill rate indicator (e.g., "8/12 specs filled") — not in MVP.

### Specification Drag-and-Drop Reorder

```
╔════════════════════════════════════════════════════════════════════════╗
║  User drags "Weight" spec from position 3 to position 1              ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Before:                                                               ║
║  ┌────────────────────────────────────────────────────────┐           ║
║  │  = │ Brand *        │ Logitech___________________│ [x] │           ║
║  │  = │ Connectivity * │ Wireless (2.4GHz)      [v] │ [x] │           ║
║  │  = │ Sensor Type    │ Optical              [v]   │ [x] │           ║
║  │  = │ Weight         │ 85g________________________│ [x] │  DRAG     ║
║  │  = │ Warranty       │ 2 Years__________________  │ [x] │           ║
║  └────────────────────────────────────────────────────────┘           ║
║                                                                        ║
║  User grabs "Weight" row by drag handle (=), drags up:                ║
║                                                                        ║
║  During drag:                                                          ║
║  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┐           ║
║    = │ Weight         │ 85g________________________│ [x]              ║
║  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┘           ║
║  ▲ DragOverlay (floating row following cursor)                        ║
║                                                                        ║
║  After:                                                                ║
║  ┌────────────────────────────────────────────────────────┐           ║
║  │  = │ Brand *        │ Logitech___________________│ [x] │  pos: 0   ║
║  │  = │ Weight         │ 85g________________________│ [x] │  pos: 1   ║
║  │  = │ Connectivity * │ Wireless (2.4GHz)      [v] │ [x] │  pos: 2   ║
║  │  = │ Sensor Type    │ Optical              [v]   │ [x] │  pos: 3   ║
║  │  = │ Warranty       │ 2 Years__________________  │ [x] │  pos: 4   ║
║  └────────────────────────────────────────────────────────┘           ║
║                                                                        ║
║  Implementation: @dnd-kit SortableContext on spec rows                 ║
║  - Each spec row is a useSortable item keyed by field index            ║
║  - onDragEnd: call form.move(fromIndex, toIndex)                       ║
║  - Recalculate position values (0, 1, 2, ...)                          ║
║  - Position is stored in spec.position field sent to API               ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### When No Category Schema Exists

```
╔════════════════════════════════════════════════════════════════════════╗
║  SPECIFICATIONS (no schema -- free-form)                               ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ┌─────────────────────────────────────────────────────────────┐      ║
║  │  = │ Name                    │ Value                    │ [x]│      ║
║  │  = │ ┌──────────────────┐   │ ┌────────────────────┐   │    │      ║
║  │    │ │ Material_________│   │ │ ABS Plastic________│   │    │      ║
║  │    │ └──────────────────┘   │ └────────────────────┘   │    │      ║
║  ├─────────────────────────────────────────────────────────────┤      ║
║  │  = │ ┌──────────────────┐   │ ┌────────────────────┐   │ [x]│      ║
║  │    │ │ Battery Life_____│   │ │ 40 hours___________│   │    │      ║
║  │    │ └──────────────────┘   │ └────────────────────┘   │    │      ║
║  └─────────────────────────────────────────────────────────────┘      ║
║                                                                        ║
║  [+ Add Specification]                                                 ║
║                                                                        ║
║  = drag handle to reorder spec position                                ║
║  [x] removes the specification row                                     ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Section 5: Gallery

Covered in detail in [05-gallery-manager.md](./05-gallery-manager.md).

---

## Section 6: Regional Pricing

Covered in detail in [06-regional-pricing.md](./06-regional-pricing.md).

---

## Form Submission

### Auto-Save (Draft)

```
╔════════════════════════════════════════════════════════════════════════╗
║  AUTO-SAVE BEHAVIOR FOR DRAFT PRODUCTS                                ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Auto-save triggers:                                                   ║
║  - Every 60 seconds if the form is dirty (has unsaved changes).        ║
║  - On blur of a major section (e.g., user scrolls past Basic Info).    ║
║  - Auto-save ONLY applies to products in draft status.                 ║
║  - Disabled once the product is submitted as "pending".                ║
║                                                                        ║
║  Auto-save saves to localStorage as a snapshot:                        ║
║  Key: "product-draft-{category_id}" (create) or                       ║
║       "product-draft-{product_id}" (edit of a draft)                   ║
║  Value: JSON snapshot of current form state + timestamp.               ║
║                                                                        ║
║  Recovery on page reload:                                              ║
║  If a localStorage snapshot exists and is newer than the server data   ║
║  (or no server data exists for create), show a recovery prompt:        ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │  Unsaved draft found from 5 minutes ago.                          │ ║
║  │                            [Restore Draft]  [Discard]             │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║                                                                        ║
║  Save indicator in sticky action bar:                                  ║
║                                                                        ║
║  IDLE:                                                                 ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │                                    [Save as Draft] [Submit]       │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║                                                                        ║
║  AUTO-SAVING:                                                          ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │  Auto-saving draft...              [Save as Draft] [Submit]       │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║  (gray text, subtle spinner)                                           ║
║                                                                        ║
║  AUTO-SAVED:                                                           ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │  Draft auto-saved at 2:45 PM       [Save as Draft] [Submit]      │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║  (green text, fades to gray after 5s)                                  ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Save Button States

```
╔════════════════════════════════════════════════════════════════════════╗
║  BUTTON INTERACTION STATES                                            ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  [Save as Draft] button lifecycle:                                     ║
║                                                                        ║
║  DEFAULT            HOVER              LOADING                         ║
║  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐         ║
║  │ Save as Draft│  │ Save as Draft│  │ ⏳ Saving...         │         ║
║  │  (blue bg)   │  │ (darker blue)│  │ (blue bg, disabled)  │         ║
║  └──────────────┘  └──────────────┘  └──────────────────────┘         ║
║                                                                        ║
║  SUCCESS (500ms)     DISABLED                                          ║
║  ┌──────────────┐  ┌──────────────────────────────────────┐           ║
║  │ ✓ Saved!     │  │ Save as Draft                        │           ║
║  │ (green flash) │  │ (gray bg, cursor not-allowed)        │           ║
║  └──────────────┘  │ Tooltip: "Select a category and      │           ║
║                     │ enter a title to save"               │           ║
║                     └──────────────────────────────────────┘           ║
║                                                                        ║
║  [Submit for Review] button lifecycle:                                 ║
║                                                                        ║
║  DEFAULT            HOVER              LOADING                         ║
║  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐         ║
║  │Submit Review  │  │Submit Review │  │ ⏳ Submitting...     │         ║
║  │  (green bg)   │  │(darker green)│  │ (green bg, disabled) │         ║
║  └──────────────┘  └──────────────┘  └──────────────────────┘         ║
║                                                                        ║
║  SUCCESS (500ms)     DISABLED                                          ║
║  ┌──────────────┐  ┌──────────────────────────────────────┐           ║
║  │ ✓ Submitted!  │  │ Submit for Review                    │           ║
║  │ (green flash) │  │ (gray bg, cursor not-allowed)        │           ║
║  └──────────────┘  │ Tooltip: "Complete all required       │           ║
║                     │ fields: category, title, 1+ SKU,     │           ║
║                     │ primary image, required specs"        │           ║
║                     └──────────────────────────────────────┘           ║
║                                                                        ║
║  Button disabled conditions:                                           ║
║  - Save as Draft: no category OR no title                              ║
║  - Submit: any required field missing (see tooltip)                    ║
║  - Both: mutation isPending (loading state)                            ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Save as Draft

```
╔════════════════════════════════════════════════════════════════════════╗
║  User clicks [Save as Draft]                                          ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Client-side validation (draft -- relaxed):                            ║
║  - shipping_category_id is required                                    ║
║  - title is required (1-255 chars)                                     ║
║  - All other fields optional                                           ║
║                                                                        ║
║  If properties exist, validate:                                        ║
║  - Each property has a name                                            ║
║  - Each property has at least 1 value                                  ║
║  - Each SKU has a price >= 0.01 and stock >= 0                         ║
║                                                                        ║
║  Build request body:                                                   ║
║  POST /api/seller/agent/seller-product/v1/seller-products              ║
║  {                                                                     ║
║    shipping_category_id: "9ce8d...",                                   ║
║    title: "Wireless Gaming Mouse...",                                  ║
║    description: "<p>...</p>",                                          ║
║    status: "draft",                                                    ║
║    source_product_id: "01KJJJD0M7E89..." | null,                      ║
║    properties: [...],                                                  ║
║    skus: [...],                                                        ║
║    specifications: [...],                                              ║
║    gallery: [...]                                                      ║
║  }                                                                     ║
║                                                                        ║
║  Response (201):                                                       ║
║  Headers: ETag: "f8c3de3d1c3e..."                                      ║
║  Body: {                                                               ║
║    "message": "Product created successfully.",                         ║
║    "data": {                                                           ║
║      "object": "SellerProduct",                                        ║
║      "id": 42,                                                         ║
║      "title": "...",                                                   ║
║      "status": "draft",                                                ║
║      "etag": "f8c3de3d1c3e...",                                        ║
║      "lock_version": 1,                                                ║
║      "created_by": 42,                                                 ║
║      "created_at": "2026-03-01T10:00:00.000000Z",                      ║
║      "updated_at": "2026-03-01T10:00:00.000000Z",                      ║
║      "properties": { "object": "...Collection", "data": [...] },      ║
║      "skus": { "object": "...Collection", "data": [...] },            ║
║      "specifications": { "object": "...Collection", "data": [...] },  ║
║      "gallery": { "object": "...Collection", "data": [...] }          ║
║    }                                                                   ║
║  }                                                                     ║
║                                                                        ║
║  On success (201):                                                     ║
║  - Store ETag from response header (needed for edit page PATCH ops)    ║
║  - Toast: "Product saved as draft"                                     ║
║  - Navigate to /products/{response.data.data.id}/edit                  ║
║  - Invalidate product list + status counts                             ║
║                                                                        ║
║  On error:                                                             ║
║  - 422: Map server errors to form fields, scroll to first error        ║
║  - 404: Toast "Category not found. Please re-select category."         ║
║         (category deleted between selection and submission)            ║
║  - 400: Toast "Failed to create product. Please try again."            ║
║         (database transaction failure)                                 ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Submit for Review

```
╔════════════════════════════════════════════════════════════════════════╗
║  User clicks [Submit for Review]                                      ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Client-side validation (pending -- strict):                           ║
║  - All draft validations PLUS:                                         ║
║  - At least 1 SKU with price and stock                                 ║
║  - At least 1 gallery image marked as primary                          ║
║  - All required specifications filled (from category schema)           ║
║  - All required properties present (from category schema)              ║
║                                                                        ║
║  Build request body (same as draft, but):                              ║
║  {                                                                     ║
║    ...all fields,                                                      ║
║    status: "pending"                                                   ║
║  }                                                                     ║
║                                                                        ║
║  On success (201):                                                     ║
║  - Toast: "Product submitted for review"                               ║
║  - Navigate to /products (list page)                                   ║
║  - Invalidate product list + status counts                             ║
║                                                                        ║
║  On error: Same as Save as Draft (422, 404, 400 handling above).       ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

> Future: Product quota tracking (e.g., "45/500 products used") — not in MVP.

### Preview Before Submit

```
╔════════════════════════════════════════════════════════════════════════╗
║  PREVIEW BEFORE SUBMISSION                                            ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  The [Submit for Review] button includes a pre-submit preview step.    ║
║  After client-side validation passes, a read-only preview modal opens  ║
║  before the actual API call.                                           ║
║                                                                        ║
║  ╔══════════════════════════════════════════════════════════════╗     ║
║  ║  Review Product Before Submission                     [x]    ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║                                                              ║     ║
║  ║  Category: Electronics > Computer Peripherals > Mouse        ║     ║
║  ║                                                              ║     ║
║  ║  Title: Wireless Gaming Mouse 2.4GHz RGB Ergonomic           ║     ║
║  ║                                                              ║     ║
║  ║  Description:                                                ║     ║
║  ║  Professional wireless gaming mouse with RGB lighting...     ║     ║
║  ║                                                              ║     ║
║  ║  Variations: 2 properties, 9 SKUs                            ║     ║
║  ║  ┌───────┬──────┬─────────┬───────┐                         ║     ║
║  ║  │ Color │ Size │ Price   │ Stock │                         ║     ║
║  ║  │ Black │ S    │ $24.99  │ 100   │                         ║     ║
║  ║  │ Black │ M    │ $24.99  │ 100   │                         ║     ║
║  ║  │ ...   │      │         │       │  (scrollable)           ║     ║
║  ║  └───────┴──────┴─────────┴───────┘                         ║     ║
║  ║                                                              ║     ║
║  ║  Specifications: 5 filled                                    ║     ║
║  ║  Gallery: 3 images, 1 video                                  ║     ║
║  ║  Primary image: mouse-front.jpg                              ║     ║
║  ║                                                              ║     ║
║  ║  Regional pricing: 2 regions configured                      ║     ║
║  ║                                                              ║     ║
║  ╠══════════════════════════════════════════════════════════════╣     ║
║  ║              [Back to Edit]  [Confirm & Submit]              ║     ║
║  ╚══════════════════════════════════════════════════════════════╝     ║
║                                                                        ║
║  - [Back to Edit] closes the modal, returns to the form.               ║
║  - [Confirm & Submit] fires the actual POST request.                   ║
║  - This is a pre-submit review, not a storefront preview.              ║
║  - All data is read-only in the modal — no inline editing.             ║
║  - Sections with validation warnings are highlighted.                  ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Source Product Pre-fill (Flow A)

When `source_product_id` is present in URL:

```
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 1: Page loads with source_product_id                            ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  URL: /products/create?source_product_id=01KJJJD0M7E89XNX0EZDNY33V6   ║
║                                                                        ║
║  ┌─────────────────────────────────────────────────────────────┐      ║
║  │  Loading source product...                                   │      ║
║  │  ┌──────────────────────────────────────────────────────┐   │      ║
║  │  │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │   │      ║
║  │  │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │   │      ║
║  │  └──────────────────────────────────────────────────────┘   │      ║
║  └─────────────────────────────────────────────────────────────┘      ║
║                                                                        ║
║  Fetch source product data (read-only, from product-service):          ║
║  GET /api/product-service/.../products/{ulid}                          ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
                                    │
                                    ▼
╔════════════════════════════════════════════════════════════════════════╗
║  STEP 2: Form pre-filled from source product                         ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Pre-fill mapping:                                                     ║
║                                                                        ║
║  Source Field                         -> Form Field                    ║
║  ──────────────────────────────────────────────────────────           ║
║  title                                -> title input                   ║
║  description                          -> rich text editor              ║
║  variation.properties[].name          -> property names                ║
║  variation.properties[].values[].name -> property value text           ║
║  variation.properties[].values[].image-> value image_url               ║
║    (NOTE: source field is "image", seller product field is "image_url")║
║  variation.skus[].price.original      -> SKU base prices               ║
║  variation.skus[].property_associations -> property_values             ║
║    (NOTE: source uses ID-based { property_id, property_value_id }.     ║
║     Must convert to positional [axisIndex, valueIndex] format:         ║
║     Map source property_id to axis index in properties array,          ║
║     map source property_value_id to value index within that axis.)     ║
║  stock                                -> SKU stock (shared initial)    ║
║  specifications[].label               -> spec name                     ║
║  specifications[].value               -> spec value                    ║
║  gallery[].url                        -> gallery image URLs            ║
║                                                                        ║
║  The seller still needs to:                                            ║
║  1. Select a shipping category (not auto-mapped from source)           ║
║  2. Review and adjust all pre-filled values                            ║
║  3. Set their own prices and stock                                     ║
║  4. Upload images to their own media center (source URLs are           ║
║     displayed as previews but need to be replaced)                     ║
║                                                                        ║
║  IMPORTANT: source_product_id is read-only after creation.             ║
║  Once the product is created with a source link, the link cannot       ║
║  be changed or removed via the edit/PATCH endpoint.                    ║
║                                                                        ║
║  Source product badge:                                                 ║
║  ┌─────────────────────────────────────────────────────────────┐      ║
║  │  Sourced from: KingBank DDR5 RAM (01KJJJD0M7...)            │      ║
║  │  Source prices are in USD. Adjust for your target regions.  │      ║
║  └─────────────────────────────────────────────────────────────┘      ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Source Product Fetch Failure

```
╔════════════════════════════════════════════════════════════════════════╗
║  ERROR HANDLING FOR SOURCE PRODUCT FETCH                              ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  When source_product_id is in the URL but fetching fails:              ║
║                                                                        ║
║  404 (source product deleted or not found):                            ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │  X  Source product not found                                      │ ║
║  │     The product may have been removed from the catalog.           │ ║
║  │     You can still create your product manually.        [Dismiss]  │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║  (red toast, auto-dismiss 8s)                                          ║
║  Fall back to manual creation: strip source_product_id from URL,       ║
║  show empty form. No redirect.                                         ║
║                                                                        ║
║  Network error (timeout, offline):                                     ║
║  ┌──────────────────────────────────────────────────────────────────┐ ║
║  │  X  Failed to load source product                                 │ ║
║  │     Check your connection.                    [Retry] [Skip]      │ ║
║  └──────────────────────────────────────────────────────────────────┘ ║
║  (red toast, persistent until dismissed)                               ║
║  [Retry] re-fetches the source product.                                ║
║  [Skip] falls back to manual creation with empty form.                 ║
║                                                                        ║
║  Invalid ULID format (malformed source_product_id):                    ║
║  Redirect to clean /products/create (strip the invalid param).         ║
║  Toast: "Invalid source product reference."                            ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

---

## Create API Request Shape

```
POST /api/seller/agent/seller-product/v1/seller-products

Content-Type: application/json

{
  "shipping_category_id": "9ce8d860-ebd0-4c76-ab0b-77e8945d096f",
  "title": "Wireless Gaming Mouse 2.4GHz RGB Ergonomic",
  "description": "<p>Professional wireless gaming mouse...</p>",
  "status": "draft",
  "source_product_id": "01KJJJD0M7E89XNX0EZDNY33V6",

  "properties": [
    {
      "name": "Color",
      "position": 0,
      "values": [
        { "value": "Black", "image_url": "https://cdn.../black.jpg", "position": 0 },
        { "value": "White", "image_url": "https://cdn.../white.jpg", "position": 1 }
      ]
    },
    {
      "name": "DPI",
      "position": 1,
      "values": [
        { "value": "800", "image_url": null, "position": 0 },
        { "value": "1600", "image_url": null, "position": 1 },
        { "value": "3200", "image_url": null, "position": 2 }
      ]
    }
  ],

  "skus": [
    {
      "price": 24.99,
      "stock": 100,
      "sku_code": "BK-800",
      "is_active": true,
      "property_values": [[0, 0], [1, 0]],
      "regional_prices": [
        {
          "region_id": 1,
          "original_price": 2499.00,
          "discount_price": null,
          "stock": 50,
          "is_active": true
        }
      ]
    }
  ],

  "specifications": [
    { "name": "Brand", "value": "Logitech", "type": "text", "position": 0 },
    { "name": "Connectivity", "value": "Wireless (2.4GHz)", "type": "select", "position": 1 }
  ],

  "gallery": [
    {
      "asset_id": "da2edd02-7e1d-4936-...",
      "url": "https://cdn.../mouse-front.jpg",
      "type": "image",
      "thumbnail_url": null,
      "duration": null,
      "position": 0,
      "is_primary": true
    },
    {
      "asset_id": "49c544b9-8d16-...",
      "url": "https://cdn.../mouse-back.jpg",
      "type": "image",
      "position": 1,
      "is_primary": false
    }
  ]
}
```

**SKU property_values mapping:**

```
property_values: [[axisIndex, valueIndex], ...]

Properties:
  [0] Color: [0]=Black, [1]=White
  [1] DPI:   [0]=800, [1]=1600, [2]=3200

SKU for "Black, 800":
  property_values: [[0, 0], [1, 0]]
                     ^Color=Black  ^DPI=800

SKU for "White, 3200":
  property_values: [[0, 1], [1, 2]]
                     ^Color=White  ^DPI=3200
```

---

## Validation Error Handling

```
╔════════════════════════════════════════════════════════════════════════╗
║  VALIDATION ERROR DISPLAY                                             ║
╠════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  Server returns 422:                                                   ║
║  {                                                                     ║
║    "message": "The given data was invalid.",                           ║
║    "errors": {                                                         ║
║      "title": ["The title field is required."],                        ║
║      "properties.0.name": [                                            ║
║        "The property 'Weight' is not defined in category schema."      ║
║      ],                                                                ║
║      "skus.2.price": ["The skus.2.price must be at least 0.01."],      ║
║      "gallery": [                                                      ║
║        "Exactly one gallery image must be marked as primary."          ║
║      ]                                                                 ║
║    }                                                                   ║
║  }                                                                     ║
║                                                                        ║
║  Frontend handling:                                                    ║
║                                                                        ║
║  1. Title field shows inline error:                                    ║
║     ┌───────────────────────────────────────────────┐                 ║
║     │ _______________________________________________│                 ║
║     └───────────────────────────────────────────────┘                 ║
║     The title field is required.  <-- red text below input             ║
║                                                                        ║
║  2. Property name shows inline error:                                  ║
║     ┌──────────────────┐                                              ║
║     │ Weight___________│                                              ║
║     └──────────────────┘                                              ║
║     Not defined in category schema. <-- red text                       ║
║                                                                        ║
║  3. SKU row highlights:                                                ║
║     │ White │ 3200 │ [0.00] │ ...                                      ║
║                      ^^^^^^^                                           ║
║                      red border + "Must be at least 0.01"              ║
║                                                                        ║
║  4. Gallery section shows banner error:                                ║
║     ┌──────────────────────────────────────────────────────────┐      ║
║     │ Exactly one gallery image must be marked as primary.      │      ║
║     └──────────────────────────────────────────────────────────┘      ║
║                                                                        ║
║  5. Page scrolls to first error section automatically.                 ║
║                                                                        ║
╚════════════════════════════════════════════════════════════════════════╝
```

### Complete Server Error Reference (Create Endpoint)

```
All possible error responses from POST /seller-products:

HTTP 404:
  "Shipping category not found."
  -> Category UUID does not exist or was deleted

HTTP 422 — Field validation:
  "The title field is required."
  "The title must not be greater than 255 characters."
  "The shipping_category_id must be a valid UUID."
  "Category must be a leaf category (no children)."
  "The description must not be greater than 65535 characters."
  "The status must be one of: draft, pending."

HTTP 422 — Properties:
  "The required property '{name}' is missing."
  "The property '{name}' is not defined in category schema."
  "The property value '{value}' is not allowed for '{name}'. Allowed: {options}."
  "Duplicate property name: '{name}'."
  "Duplicate value '{value}' in property '{name}'."
  "Maximum 3 properties allowed."
  "Maximum 50 values per property."

HTTP 422 — SKUs:
  "The skus.*.price must be at least 0.01."
  "The skus.*.price must not be greater than 9999999999.99."
  "The skus.*.stock must be at least 0."
  "Duplicate SKU: two SKUs reference the same combination of property values."
  "The sku_code '{code}' is already used by another SKU in this product."
  "Maximum 100 SKUs allowed."
  "SKU property_values references are inconsistent."

HTTP 422 — Specifications:
  "The required specification '{name}' is missing."
  "The specification '{name}' requires one of: {options}. Got: '{value}'."
  "Duplicate specification name: '{name}'."
  "Maximum 50 specifications allowed."

HTTP 422 — Gallery:
  "Exactly one gallery image must be marked as primary."
  "is_primary can only be set on type=image entries, not videos."
  "Gallery video requires a thumbnail_url."
  "The asset '{asset_id}' was not found."
  "Maximum 20 gallery items allowed."

HTTP 422 — Regional pricing:
  "Region ID {id} does not exist."
  "Region ID {id} is already assigned to this SKU."

HTTP 400:
  "Failed to create product. Please try again."
  -> Database transaction failure (rare)

Error field path mapping (server key -> form field):
  "title"                        -> form.title
  "description"                  -> form.description
  "shipping_category_id"         -> form.shipping_category_id
  "properties.{i}.name"         -> form.properties[i].name
  "properties.{i}.values.{j}"   -> form.properties[i].values[j].value
  "skus.{i}.price"              -> SKU table row i, price cell
  "skus.{i}.stock"              -> SKU table row i, stock cell
  "skus.{i}.sku_code"           -> SKU table row i, sku_code cell
  "specifications.{i}.name"     -> form.specifications[i].name
  "specifications.{i}.value"    -> form.specifications[i].value
  "gallery"                      -> gallery section banner
  "gallery.{i}.url"             -> gallery item i
  "gallery.{i}.asset_id"        -> gallery item i
```

---

## Form State Management

```
Form state managed by React Hook Form + Zod schema.

Zod schema structure:
{
  shipping_category_id: z.string().uuid(),
  title: z.string().min(1).max(255),
  description: z.string().max(65535).nullable(),
  status: z.enum(["draft", "pending"]).optional().default("draft"),
    // Backend default is "draft" when omitted. Both are valid.
  source_product_id: z.string().max(30).nullable(),
  properties: z.array(PropertySchema).max(3),
  skus: z.array(SkuSchema).max(100),
  specifications: z.array(SpecSchema).max(50),
  gallery: z.array(GallerySchema).max(20),
}

PropertySchema:
{
  name: z.string().min(1).max(100),
    // min(1) required -- backend rejects empty property names
  position: z.number().int().min(0),
  values: z.array({
    value: z.string().min(1).max(100),
    image_url: z.string().url().max(500).nullable(),
    position: z.number().int().min(0),
  }).min(1).max(50),
}

SkuSchema:
{
  price: z.number().min(0.01).max(9999999999.99),
    // min is 0.01, NOT 0 -- backend rejects price=0
    // API returns price as string (DECIMAL). Form uses number.
    // parseFloat() on load. See 07 "Type Conversion" section.
  stock: z.number().int().min(0),
  sku_code: z.string().max(50).nullable(),
    // Must be unique per product (cross-SKU check via superRefine)
  is_active: z.boolean().default(true),
  property_values: z.array(z.tuple([z.number(), z.number()])),
  regional_prices: z.array(RegionalPriceSchema).optional().default([]),
}

RegionalPriceSchema:
{
  region_id: z.number().int().positive(),
    // Must reference a valid region in Core module (exists:regions,id)
  original_price: z.number().min(0.01).max(9999999999.99),
  discount_price: z.number().min(0.01).max(9999999999.99).nullable(),
    // If set, should not exceed original_price (business rule)
  stock: z.number().int().min(0),
  is_active: z.boolean().default(true),
}

SpecSchema:
{
  name: z.string().min(1).max(100),
  value: z.string().min(1).max(500),
  type: z.enum(["text", "select"]).default("text"),
  position: z.number().int().min(0),
}

GallerySchema:
{
  asset_id: z.string().uuid().nullable().optional(),
    // If provided, must exist in Storage and belong to same agent company
  url: z.string().url().max(500),
  type: z.enum(["image", "video"]).default("image"),
  thumbnail_url: z.string().url().max(500).nullable(),
    // Required when type="video"
  duration: z.number().int().min(0).nullable(),
    // Optional, for video only
  position: z.number().int().min(0),
  is_primary: z.boolean().default(false),
    // Exactly one image must be primary. Videos cannot be primary.
}

Cross-field validation (via .superRefine() on the root schema):
- Exactly one gallery item with type="image" must have is_primary=true
- No duplicate property names
- No duplicate values within a property
- No duplicate specification names
- No duplicate SKU combinations (same set of property_values)
- No duplicate sku_code values within the product (where non-null)
- No duplicate region_id within a SKU's regional_prices
- When type="video": thumbnail_url is required, is_primary must be false

TanStack Query mutation:

useMutation({
  mutationFn: (data) => api.product.post('/seller-products', data),
  onSuccess: (response) => {
    // response shape: { message, data: { id, etag, lock_version, ... } }
    queryClient.invalidateQueries({ queryKey: ['seller-products'] })

    // Store ETag for the edit page (needed for PATCH If-Match header)
    const etag = response.headers?.etag || response.data.data.etag
    queryClient.setQueryData(
      ['seller-products', 'detail', response.data.data.id],
      response.data.data
    )

    toast.success('Product saved')
    router.push(`/products/${response.data.data.id}/edit`)
  },
  onError: (error) => {
    if (error.status === 422) {
      // Map validation errors to form fields via setError()
      mapServerErrors(error.errors, form.setError)
      scrollToFirstError()
    } else if (error.status === 404) {
      // Category was deleted between selection and submission
      toast.error('Category not found. Please re-select category.')
    } else if (error.status === 400) {
      // Database transaction failure
      toast.error('Failed to create product. Please try again.')
    } else {
      toast.error('An unexpected error occurred.')
    }
  }
})
```
