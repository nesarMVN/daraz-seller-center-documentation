# Seller Product Architecture

The Seller module enables sellers and agent companies to create, manage, and list their own products on the MoveOn platform. It covers the full product lifecycle — from draft creation (manual or sourced from vendor catalog) through properties, SKU matrices, pricing, specifications, gallery management, and status transitions. The module supports regional pricing and stock per SKU across multiple regions (BD, IN, AE, US, EU, AU). All seller product entities use BIGINT auto-increment primary keys and ID-based JSON Patch addressing for updates.

## Table of Contents

1. [Business Requirements](./01-business-requirements.md) — Requirements, constraints, user stories
2. [Technical Overview](./02-technical-overview.md) — Module overview, module relationships, architecture diagrams
3. [Database Schema](./03-database-schema.md) — All 8 tables, indexes, foreign keys, design decisions, enums
4. [API Specification](./04-api-specification.md) — Routes, all 8 API workflows with full request/response
5. [Validation Rules](./05-validation-rules.md) — All request validation classes
6. [Resource Definitions](./06-resource-definitions.md) — All resource shape definitions
7. [Error Reference](./07-error-reference.md) — All error cases by endpoint
8. [Category Schema Integration](./08-category-schema.md) — Schema integration, validation rules from schema
9. [Source Product Linking](./09-source-product-linking.md) — Source product linking, pre-fill mapping
10. [Implementation Notes](./10-implementation-notes.md) — Repo pattern, transactions, ETag, JSON Patch, traits, events
11. [Files to Create](./11-files-to-create.md) — File tree listing
12. [Implementation Conventions](./12-implementation-conventions.md) — Repository, service, enum, interface, and provider patterns

---

_v3.1 | 2026-03-01_
