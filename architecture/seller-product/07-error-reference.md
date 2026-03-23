# Error Reference

## List Products (GET /)

| HTTP | Scenario                                        | Message                                                 |
| ---- | ----------------------------------------------- | ------------------------------------------------------- |
| 422  | Invalid status value                            | The status must be one of: draft, pending, active, inactive, violation, deleted. |
| 422  | Invalid shipping_category_id format             | The shipping_category_id must be a valid UUID.          |
| 422  | Invalid sort_by value                           | The sort_by must be one of: title, created_at, updated_at, price. |
| 422  | Invalid sort_order value                        | The sort_order must be one of: asc, desc.               |
| 422  | per_page exceeds max                            | The per_page must not be greater than 100.              |
| 422  | Invalid region_id format                        | The region_id must be a valid integer.                  |

## Show Product (GET /{id})

| HTTP | Scenario                                        | Message                                                 |
| ---- | ----------------------------------------------- | ------------------------------------------------------- |
| 404  | Product not found or unauthorized               | Product not found.                                      |
| 422  | Invalid expand[] value                          | The expand must be one of: properties, skus, specifications, gallery, category, regional_prices. |
| 422  | Invalid region_id format                        | The region_id must be a valid integer.                  |

## Create Product (POST /)

| HTTP | Scenario                                        | Message                                                 |
| ---- | ----------------------------------------------- | ------------------------------------------------------- |
| 404  | shipping_category_id not found                  | Shipping category not found.                            |
| 422  | Category is not a leaf node                     | Category must be a leaf category (no children).         |
| 422  | Title empty or too long                         | The title field is required. / The title must not exceed 255 characters. |
| 422  | Description too long                            | The description must not exceed 65535 characters.       |
| 422  | Properties exceed max 3                         | The properties must not have more than 3 items.         |
| 422  | Property values exceed max 50 per property      | The property '{name}' must not have more than 50 values. |
| 422  | Property name not in schema                     | The property '{name}' is not defined in the category schema. |
| 422  | Required property missing                       | The required property '{name}' is missing.              |
| 422  | Property value not in schema options             | The value '{value}' is not a valid option for '{name}'. |
| 422  | Duplicate property names                        | Duplicate property name: '{name}'.                      |
| 422  | Duplicate property values within property       | Duplicate value '{value}' in property '{name}'.         |
| 422  | SKUs exceed max 100                             | The skus must not have more than 100 items.             |
| 422  | SKU property_values inconsistent                | Each SKU must reference exactly one value from each property. |
| 422  | Duplicate SKU combination                       | Duplicate SKU: two SKUs reference the same combination of property values. |
| 422  | Duplicate sku_code within product               | The sku_code '{code}' is already used by another SKU in this product. |
| 422  | SKU price too low                               | The skus.{i}.price must be at least 0.01.              |
| 422  | Required spec missing                           | The required specification '{name}' is missing.        |
| 422  | Select spec value not in options                | The specification '{name}' requires one of: {options}. Got: '{value}'. |
| 422  | Specifications exceed max 50                    | The specifications must not have more than 50 items.   |
| 422  | Duplicate specification names                   | Duplicate specification name: '{name}'.                |
| 422  | Gallery exceeds max 20                          | The gallery must not have more than 20 items.          |
| 422  | Zero or multiple primary gallery images         | Exactly one gallery image must be marked as primary.   |
| 422  | Gallery asset_id not found                      | The asset '{asset_id}' was not found.                  |
| 422  | Video missing thumbnail_url                     | Gallery video must include a thumbnail_url.            |
| 422  | is_primary set on video                         | Only images can be marked as primary.                  |
| 422  | Regional price region_id not found              | Region ID {id} does not exist.                         |
| 422  | Duplicate region_id within a SKU                | Region ID {id} is already assigned to this SKU.        |
| 400  | Database transaction failure                    | Failed to create product. Please try again.            |

## Update Product (PATCH /{id}) — JSON Patch

| HTTP | Scenario                                                     | Message                                                                     |
| ---- | ------------------------------------------------------------ | --------------------------------------------------------------------------- |
| 404  | Product not found or unauthorized                            | Product not found.                                                          |
| 409  | `test` operation failed                                      | Test failed at '{path}': expected {expected}, got {actual}.                 |
| 412  | `If-Match` etag does not match current state                 | Precondition failed. The resource has been modified. Re-fetch and retry.    |
| 415  | Content-Type is not `application/json-patch+json`            | Unsupported media type. Expected: application/json-patch+json.             |
| 422  | Patch document is not a JSON array                           | Patch document must be a JSON array of operations.                         |
| 422  | Operation missing `op` or `path`                             | Operation at index {i} missing required field: {field}.                    |
| 422  | `op` not one of: add, remove, replace, test                  | Operation at index {i}: unsupported op '{op}'.                             |
| 422  | Path does not match any addressable target                   | Path '{path}' is not a valid target for this resource.                     |
| 422  | `replace` target does not exist                              | Cannot replace '{path}' — ID not found.                                    |
| 422  | `remove` target does not exist                               | Cannot remove '{path}' — ID not found.                                     |
| 422  | ID not found in collection                                   | ID {id} not found in {collection}.                                         |
| 422  | Value type mismatch (e.g. string where number expected)      | Invalid value at '{path}': expected {type}.                                |
| 422  | Invalid status transition                                    | Cannot transition from '{from}' to '{to}'.                                 |
| 422  | `add` on SKU but property_value_ids invalid                  | The SKU references property values that do not exist: {ids}.               |
| 422  | `_tempId` referenced in `property_value_refs` not found      | The temporary reference '{ref}' was not found in any add operation.        |
| 422  | Post-op: total properties > 3                                | Product cannot have more than 3 properties. Current: {n} after changes.    |
| 422  | Post-op: property values per property > 50                   | Property '{name}' cannot have more than 50 values. Current: {n} after changes. |
| 422  | Post-op: duplicate sku_code within product                   | The sku_code '{code}' is already used by another SKU in this product.      |
| 422  | Post-op: total SKUs > 100                                    | Product cannot have more than 100 SKUs. Current: {n} after changes.        |
| 422  | Post-op: total specs > 50                                    | Product cannot have more than 50 specifications. Current: {n} after changes.|
| 422  | Post-op: total gallery > 20                                  | Product cannot have more than 20 gallery items. Current: {n} after changes. |
| 422  | Post-op: zero or multiple primary gallery images             | Exactly one gallery image must be marked as primary.                       |
| 422  | `replace` is_primary=true on video entry                     | Only images can be marked as primary.                                      |
| 422  | `replace` type to video on primary entry                     | Cannot change type to video on primary gallery item. Reassign primary first. |
| 422  | `replace` type to video without thumbnail_url                | Cannot change type to video without a thumbnail_url. Set thumbnail_url first. |
| 422  | `add` gallery video without thumbnail_url                    | Gallery video must include a thumbnail_url.                                |
| 422  | `add` gallery video with is_primary=true                     | Only images can be marked as primary.                                      |
| 422  | Post-op: is_primary on a video entry                         | Gallery item '{id}' is a video but has is_primary=true.                    |
| 422  | Post-op: duplicate property names                            | Duplicate property name: '{name}'.                                         |
| 422  | Post-op: duplicate spec names                                | Duplicate specification name: '{name}'.                                    |
| 422  | Post-op: orphaned SKU after value removal                    | SKU '{id}' has incomplete property associations after value removal.        |
| 422  | Post-op: schema violation after category change              | Property/specification '{name}' does not conform to the new category schema. |
| 422  | `add` regional price — region_id not found                   | Region ID {id} does not exist.                                              |
| 422  | `add` regional price — duplicate region for SKU              | Region ID {id} is already assigned to this SKU.                            |
| 422  | `add` regional price — missing required fields               | Regional price must include region_id, original_price, and stock.          |
| 422  | `replace` regional price — invalid value                     | Regional original price must be at least 0.01.                             |
| 428  | `If-Match` header missing                                    | If-Match header is required for PATCH operations.                          |
| 400  | Database transaction failure                                 | Failed to update product. Please try again.                                |

## Delete Product (DELETE /{id})

| HTTP | Scenario                                        | Message                                                 |
| ---- | ----------------------------------------------- | ------------------------------------------------------- |
| 404  | Product not found or unauthorized               | Product not found.                                      |

## Bulk Status Update (POST /bulk-status)

| HTTP | Scenario                                        | Message                                                 |
| ---- | ----------------------------------------------- | ------------------------------------------------------- |
| 403  | Some products not found or unauthorized         | Some products not found or unauthorized.                |
| 422  | Empty product_ids                               | The product_ids field is required.                      |
| 422  | product_ids exceeds 50                          | The product_ids must not have more than 50 items.      |
| 422  | Invalid target status for bulk                  | The status must be one of: active, inactive, deleted.  |
| 422  | Invalid ID in product_ids                       | The product_ids.{i} must be a valid integer.           |

## Bulk Delete (POST /bulk-delete)

| HTTP | Scenario                                        | Message                                                 |
| ---- | ----------------------------------------------- | ------------------------------------------------------- |
| 403  | Some products not found or unauthorized         | Some products not found or unauthorized.                |
| 422  | Empty product_ids                               | The product_ids field is required.                      |
| 422  | product_ids exceeds 50                          | The product_ids must not have more than 50 items.      |
| 422  | Invalid ID in product_ids                       | The product_ids.{i} must be a valid integer.           |
