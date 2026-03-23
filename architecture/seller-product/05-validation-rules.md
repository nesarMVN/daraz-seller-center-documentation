# Validation Rules

## AgentSellerProductIndexRequest

| Field                 | Rules                                                    | Default    |
| --------------------- | -------------------------------------------------------- | ---------- |
| response_type         | nullable, in:ResponseTypEnum                             | regular    |
| status                | nullable, in:SellerProductStatusEnum                     | -          |
| shipping_category_id  | nullable, uuid                                           | -          |
| title                 | nullable, string, max:255                                | -          |
| sort_by               | nullable, in:SellerProductSortFieldsEnum                 | created_at |
| sort_order            | nullable, in:SortOrderEnum                               | desc       |
| page                  | nullable, integer, min:1                                 | 1          |
| per_page              | nullable, integer, min:1, max:100                        | 20         |
| region_id             | nullable, integer                                        | -          |

## AgentSellerProductShowRequest

| Field          | Rules                                        | Default |
| -------------- | -------------------------------------------- | ------- |
| response_type  | nullable, in:ResponseTypEnum                 | regular |
| expand         | nullable, array                              | -       |
| expand.*       | nullable, in:SellerProductExpandEnum         | -       |
| region_id      | nullable, integer                            | -       |

## AgentSellerProductStoreRequest

| Field                                   | Rules                                                         |
| --------------------------------------- | ------------------------------------------------------------- |
| shipping_category_id                    | required, uuid                                                |
| title                                   | required, string, min:1, max:255                              |
| description                             | nullable, string, max:65535                                   |
| status                                  | nullable, in:draft,pending                                    |
| source_product_id                       | nullable, string, max:30                                      |
| properties                              | nullable, array, max:3                                        |
| properties.*.name                       | required_with:properties, string, max:100                     |
| properties.*.position                   | required_with:properties, integer, min:0                      |
| properties.*.values                     | required_with:properties, array, min:1, max:50                |
| properties.*.values.*.value             | required, string, max:100                                     |
| properties.*.values.*.image_url         | nullable, string, url, max:500                                |
| properties.*.values.*.position          | required, integer, min:0                                      |
| skus                                    | nullable, array, max:100                                      |
| skus.*.price                            | required_with:skus, numeric, min:0.01, max:9999999999.99      |
| skus.*.stock                            | required_with:skus, integer, min:0                            |
| skus.*.sku_code                         | nullable, string, max:50                                      |
| skus.*.is_active                        | nullable, boolean                                             |
| skus.*.property_values                  | required_with:skus, array                                     |
| skus.*.property_values.*                | required, array, size:2                                       |
| skus.*.property_values.*.*              | required, integer, min:0                                      |
| specifications                          | nullable, array, max:50                                       |
| specifications.*.name                   | required_with:specifications, string, max:100                 |
| specifications.*.value                  | required_with:specifications, string, max:500                 |
| specifications.*.type                   | nullable, in:text,select                                      |
| specifications.*.position               | required_with:specifications, integer, min:0                  |
| gallery                                 | nullable, array, max:20                                       |
| gallery.*.asset_id                      | nullable, uuid                                                |
| gallery.*.url                           | required_with:gallery, string, url, max:500                   |
| gallery.*.position                      | required_with:gallery, integer, min:0                         |
| gallery.*.type                          | nullable, in:image,video                                      |
| gallery.*.thumbnail_url                 | nullable, required_if:gallery.*.type,video, string, url, max:500 |
| gallery.*.duration                      | nullable, integer, min:0                                      |
| gallery.*.is_primary                    | nullable, boolean                                             |
| skus.*.regional_prices                  | nullable, array                                               |
| skus.*.regional_prices.*.region_id      | required_with:skus.*.regional_prices, integer, exists:regions,id |
| skus.*.regional_prices.*.original_price | required_with:skus.*.regional_prices, numeric, min:0.01, max:9999999999.99 |
| skus.*.regional_prices.*.discount_price | nullable, numeric, min:0.01, max:9999999999.99                |
| skus.*.regional_prices.*.stock          | required_with:skus.*.regional_prices, integer, min:0          |
| skus.*.regional_prices.*.is_active      | nullable, boolean                                             |

## AgentSellerProductUpdateRequest (JSON Patch — RFC 6902)

This request class does not use Laravel's standard form request validation for the patch body. Instead, it validates the structural requirements of a JSON Patch document and defers semantic validation (value types, ranges, path validity) to the `JsonPatchProcessor` service.

**Header validation:**

| Header       | Rules                                                                    |
| ------------ | ------------------------------------------------------------------------ |
| Content-Type | Must be `application/json-patch+json`. Returns 415 if wrong.            |
| If-Match     | Required. Must be a quoted string (ETag). Returns 428 if missing.       |

**Body validation (structural):**

| Field                   | Rules                                                              |
| ----------------------- | ------------------------------------------------------------------ |
| (root)                  | required, array — the patch document must be a JSON array          |
| *.op                    | required, string, in:add,remove,replace,test                      |
| *.path                  | required, string, starts_with:/ — valid JSON Pointer (RFC 6901)   |
| *.value                 | required_if:*.op,add,replace,test — the value to set or test      |

**Semantic validation (in `JsonPatchProcessor`):**

After structural validation passes, the `JsonPatchProcessor` validates each operation:

| Check                                 | Error                                                                |
| ------------------------------------- | -------------------------------------------------------------------- |
| Path matches an addressable path      | 422: "Path '/foo/bar' is not a valid target for this resource."      |
| `replace` target exists               | 422: "Cannot replace '/skus/42/price' — ID 42 not found."           |
| `remove` target exists                | 422: "Cannot remove '/specifications/7' — ID 7 not found."         |
| ID not found in collection            | 422: "ID {id} not found in {collection} (available IDs: {ids})."   |
| `replace` on `/title`: string, 1-255  | 422: "Title must be a string between 1 and 255 characters."         |
| `replace` on `/status`: valid enum    | 422: "Status must be one of: draft, pending, active, inactive, violation, deleted." |
| `replace` on `/skus/{id}/price`: numeric, ≥ 0.01 | 422: "SKU price must be at least 0.01."                  |
| `replace` on `/skus/{id}/stock`: integer, ≥ 0 | 422: "SKU stock must be a non-negative integer."             |
| `add` on `/properties/-`: object has name, position, values | 422: "Property must include name, position, and values." |
| `add` on `/skus/-`: object has price, stock, property_value_ids | 422: "SKU must include price, stock, and property_value_ids." |
| `add` on `/gallery/-`: object has url, position | 422: "Gallery item must include url and position."           |
| `add` on `/gallery/-`: if type=video, thumbnail_url required | 422: "Gallery video must include a thumbnail_url."     |
| `add` on `/gallery/-`: if type=video, is_primary must not be true | 422: "Only images can be marked as primary."        |
| `replace` on `/gallery/{id}/type`: must be image or video | 422: "Gallery type must be one of: image, video."    |
| `replace` on `/gallery/{id}/type` to video: thumbnail_url must be non-null | 422: "Cannot change type to video without a thumbnail_url. Set thumbnail_url first." |
| `replace` on `/gallery/{id}/type` to video: is_primary must be false | 422: "Cannot change type to video on primary gallery item. Reassign primary first." |
| `replace` on `/gallery/{id}/thumbnail_url`: string, url, max 500 or null | 422: "Gallery thumbnail URL must be a valid URL (max 500 chars) or null." |
| `replace` on `/gallery/{id}/duration`: integer, min 0 or null | 422: "Gallery duration must be a non-negative integer or null." |
| `replace` on `/gallery/{id}/is_primary` = true: target must be type=image | 422: "Only images can be marked as primary." |
| `add` on `/specifications/-`: object has name, value, position | 422: "Specification must include name, value, and position." |
| `add` on `/properties/{id}/values/-` with `_tempId`: string, optional | Used for intra-patch reference resolution only.              |
| `add` on `/skus/-` with `property_value_refs`: each ref matches a `_tempId` in an earlier `add` | 422: "The temporary reference '{ref}' was not found in any add operation." |
| `add` on `/skus/{id}/regional_prices/-`: object has region_id, original_price, stock | 422: "Regional price must include region_id, original_price, and stock." |
| `add` on `/skus/{id}/regional_prices/-`: region_id exists in Core regions | 422: "Region ID {id} does not exist." |
| `add` on `/skus/{id}/regional_prices/-`: no duplicate region_id per SKU | 422: "Region ID {id} is already assigned to this SKU." |
| `replace` on `/skus/{id}/regional_prices/{id}/original_price`: numeric, ≥ 0.01 | 422: "Regional original price must be at least 0.01." |
| `replace` on `/skus/{id}/regional_prices/{id}/discount_price`: numeric, ≥ 0.01 or null | 422: "Regional discount price must be at least 0.01 or null." |
| `replace` on `/skus/{id}/regional_prices/{id}/stock`: integer, ≥ 0 | 422: "Regional stock must be a non-negative integer." |

## AgentSellerProductDestroyRequest

No request body. Path parameter `id` is validated as integer by Laravel route model binding.

## AgentSellerProductBulkStatusRequest

| Field           | Rules                                                  |
| --------------- | ------------------------------------------------------ |
| product_ids     | required, array, min:1, max:50                         |
| product_ids.*   | required, integer                                      |
| status          | required, in:active,inactive,deleted                   |

## AgentSellerProductBulkDeleteRequest

| Field           | Rules                                                  |
| --------------- | ------------------------------------------------------ |
| product_ids     | required, array, min:1, max:50                         |
| product_ids.*   | required, integer                                      |

## AgentSellerProductStatusCountsRequest

No parameters. Authentication middleware resolves `owner_type` and `owner_id`.
