# Business Requirements

## What We Want To Achieve

Let sellers create, manage, and list their own products within the MoveOn platform. A seller product is a business entity distinct from the read-only source catalog (product-service). Sellers set their own pricing, choose which properties to offer, assign shipping categories, upload images and videos, and manage a status lifecycle from draft through active listing.

## Target Users

- **Sellers (via Agent Company):** Create and manage product listings through an agent company
- **Direct Sellers:** Independent sellers who manage products under their own user account
- **Agent Companies:** Manage products on behalf of multiple sellers

## Features Covered

| Feature                    | Description                                                                                         |
| -------------------------- | --------------------------------------------------------------------------------------------------- |
| **Create Product**         | Create a new product with category, properties, SKUs, specifications, and gallery                   |
| **Update Product**         | Edit any product field — title, description, pricing, properties, SKUs, specs, gallery              |
| **Delete Product**         | Soft delete a product (status change to deleted)                                                    |
| **List Products**          | Paginated list with status tab filtering, search, category filter, sort                             |
| **View Product**           | Single product with all relations (properties, SKUs, specs, gallery)                                |
| **Bulk Status Update**     | Activate, deactivate, or delete multiple products at once                                           |
| **Bulk Delete**            | Soft delete multiple products in a single operation                                                 |
| **Status Counts**          | Return product counts grouped by status for tab badges                                              |
| **Source Product Linking**  | Optionally link a seller product to a source product from product-service for data pre-fill         |
| **Category Schema Validation** | Validate submitted properties and specifications against the category's attribute schema         |
| **Regional Pricing**           | Set per-region pricing, stock, and availability for each SKU across supported regions (BD, IN, AE, US, EU, AU) |

## Constraints

| Constraint                | Value                | Reason                                                  |
| ------------------------- | -------------------- | ------------------------------------------------------- |
| Max title length          | 255 characters       | UI display and SEO constraints                          |
| Max description length    | 65,535 characters    | MySQL TEXT column, sufficient for rich HTML              |
| Max gallery items         | 20 per product       | Combined images and videos; performance and storage budget |
| Max properties per product| 3 axes               | UX complexity — more than 3 axes creates unwieldy SKU matrices |
| Max SKUs per product      | 100                  | Prevent combinatorial explosion of property values      |
| Max specifications        | 50 per product       | Category schemas define far fewer, but allow headroom   |
| Max property values per property | 50             | Prevents unwieldy UI and excessive payload size         |
| Min price                 | 0.01                 | No free products                                        |
| Max regions per SKU       | 6                    | Number of supported regions (BD, IN, AE, US, EU, AU)    |

## User Stories

1. **As a seller**, I want to create a product by selecting a shipping category so the system loads the correct property and specification schema
2. **As a seller**, I want to source a product from the vendor catalog so my listing pre-fills with source data
3. **As a seller**, I want to create a product from scratch without linking to any source
4. **As a seller**, I want to define product properties (e.g. Size, Color) and their values so the system generates a SKU matrix
5. **As a seller**, I want to set price and stock per SKU so buyers see accurate availability
6. **As a seller**, I want to fill specification fields defined by the category schema
7. **As a seller**, I want to upload and reorder product gallery images and videos
8. **As a seller**, I want to save incomplete products as drafts and finish them later
9. **As a seller**, I want to list my products filtered by status tabs (All, Active, Inactive, Draft, Pending, Violation, Deleted)
10. **As a seller**, I want to search products by name and filter by category
11. **As a seller**, I want to bulk activate, deactivate, or delete products
12. **As a seller**, I want to see product counts per status tab so I know my inventory breakdown
13. **As a seller**, I want to set different prices and stock levels per region so I can sell across multiple markets with region-specific pricing
14. **As a seller**, I want to hide specific SKUs in certain regions so I can control availability per market
