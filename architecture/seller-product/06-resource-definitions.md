# Resource Definitions

## SellerProductAgentResource

**Minimal (response_type = minimal):**

| Field       | Source                                                          |
| ----------- | --------------------------------------------------------------- |
| object      | SellerProductMinimal                                            |
| id          | id                                                              |
| title       | title                                                           |
| status      | status                                                          |
| shipping_category_id | shipping_category_id                                   |
| image       | Primary gallery image URL (first where is_primary = true)       |
| price_range | { min: min_price, max: max_price } (denormalized columns)       |
| sku_count   | Count of SKUs                                                   |
| created_at  | created_at                                                      |
| updated_at  | updated_at                                                      |

**Regular (default):**

| Field                 | Source                            | Expandable                     |
| --------------------- | --------------------------------- | ------------------------------ |
| object                | SellerProduct                     | -                              |
| id                    | id                                | -                              |
| title                 | title                             | -                              |
| description           | description                       | -                              |
| status                | status                            | -                              |
| shipping_category_id  | shipping_category_id              | -                              |
| source_product_id     | source_product_id                 | -                              |
| created_by            | created_by                        | -                              |
| etag                  | etag                              | -                              |
| lock_version          | lock_version                      | -                              |
| created_at            | created_at                        | -                              |
| updated_at            | updated_at                        | -                              |
| properties            | whenLoaded                        | expand[]=properties            |
| skus                  | whenLoaded                        | expand[]=skus                  |
| specifications        | whenLoaded                        | expand[]=specifications        |
| gallery               | whenLoaded                        | expand[]=gallery               |
| category              | whenLoaded                        | expand[]=category              |

## SellerProductPropertyAgentResource

| Field    | Source      |
| -------- | ----------- |
| object   | SellerProductProperty |
| id       | id          |
| name     | name        |
| position | position    |
| values   | SellerProductPropertyValueAgentResource::collection(values) |

## SellerProductPropertyValueAgentResource

| Field     | Source    |
| --------- | -------- |
| object    | SellerProductPropertyValue |
| id        | id       |
| value     | value    |
| image_url | image_url |
| position  | position |

## SellerProductSkuAgentResource

| Field        | Source      |
| ------------ | ----------- |
| object       | SellerProductSku |
| id           | id          |
| sku_code     | sku_code    |
| price        | price       |
| stock        | stock       |
| is_active    | is_active   |
| associations | SellerProductSkuAssociationAgentResource::collection(skuAssociations) |
| regional_prices | whenLoaded — SellerProductSkuRegionPriceAgentResource::collection(skuRegions) |

## SellerProductSkuAssociationAgentResource

| Field              | Source                                     |
| ------------------ | ------------------------------------------ |
| object             | SellerProductSkuAssociation                |
| id                 | id                                         |
| property_id        | propertyValue.property_id                  |
| property_value_id  | property_value_id                          |
| property           | Nested object from propertyValue.property   |
| property.id        | propertyValue.property.id                  |
| property.name      | propertyValue.property.name                |
| property.position  | propertyValue.property.position            |
| property_value     | Nested object from propertyValue            |
| property_value.id  | propertyValue.id                           |
| property_value.value | propertyValue.value                      |
| property_value.image_url | propertyValue.image_url              |
| property_value.position | propertyValue.position                |

## SellerProductSpecificationAgentResource

| Field    | Source      |
| -------- | ----------- |
| object   | SellerProductSpecification |
| id       | id          |
| name     | name        |
| value    | value       |
| type     | type        |
| position | position    |

## SellerProductGalleryAgentResource

| Field         | Source                  |
| ------------- | ----------------------- |
| object        | SellerProductGalleryItem |
| id            | id                      |
| asset_id      | asset_id                |
| url           | url                     |
| type          | type                    |
| thumbnail_url | thumbnail_url           |
| duration      | duration                |
| position      | position                |
| is_primary    | is_primary              |

## SellerProductSkuRegionPriceAgentResource

| Field          | Source         |
| -------------- | -------------- |
| object         | SellerProductSkuRegionPrice |
| id             | id             |
| region_id      | region_id      |
| original_price | original_price |
| discount_price | discount_price |
| stock          | stock          |
| is_active      | is_active      |

