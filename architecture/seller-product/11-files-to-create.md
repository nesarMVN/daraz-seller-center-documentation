# Files to Create

```
modules/MoveOn/Seller/src/
├── Contracts/
│   ├── SellerProduct.php
│   ├── SellerProductProperty.php
│   ├── SellerProductPropertyValue.php
│   ├── SellerProductSku.php
│   ├── SellerProductSkuAssociation.php
│   ├── SellerProductSpecification.php
│   ├── SellerProductGallery.php
│   └── SellerProductSkuRegionPrice.php
│
├── Database/
│   └── Migrations/
│       ├── 2026_03_01_000001_create_seller_products_table.php
│       ├── 2026_03_01_000002_create_seller_product_properties_table.php
│       ├── 2026_03_01_000003_create_seller_product_property_values_table.php
│       ├── 2026_03_01_000004_create_seller_product_skus_table.php
│       ├── 2026_03_01_000005_create_seller_product_sku_associations_table.php
│       ├── 2026_03_01_000006_create_seller_product_specifications_table.php
│       ├── 2026_03_01_000007_create_seller_product_gallery_table.php
│       └── 2026_03_01_000008_create_seller_product_sku_regions_table.php
│
├── Enum/
│   ├── SellerProductFieldsEnum.php
│   ├── SellerProductFiltersEnum.php
│   ├── SellerProductExpandEnum.php
│   ├── SellerProductSortFieldsEnum.php
│   ├── SellerProductStatusEnum.php
│   └── ResourceObject.php
│
├── Exception/
│   ├── SellerProductNotFoundException.php
│   ├── SellerProductCreateException.php
│   ├── SellerProductUpdateException.php
│   ├── SellerProductDeleteException.php
│   ├── SellerProductStatusTransitionException.php
│   ├── CategorySchemaValidationException.php
│   ├── EtagMismatchException.php
│   ├── JsonPatchException.php
│   └── SkuLimitExceededException.php
│
├── Http/
│   ├── agent-routes.php
│   ├── Controllers/
│   │   └── Agent/
│   │       └── V1/
│   │           └── SellerProductAgentController.php
│   ├── Requests/
│   │   ├── AgentSellerProductIndexRequest.php
│   │   ├── AgentSellerProductShowRequest.php
│   │   ├── AgentSellerProductStoreRequest.php
│   │   ├── AgentSellerProductUpdateRequest.php
│   │   ├── AgentSellerProductDestroyRequest.php
│   │   ├── AgentSellerProductBulkStatusRequest.php
│   │   ├── AgentSellerProductBulkDeleteRequest.php
│   │   └── AgentSellerProductStatusCountsRequest.php
│   └── Resources/
│       └── V1/
│           └── Agent/
│               ├── SellerProductAgentResource.php
│               ├── SellerProductAgentCollectionResource.php
│               ├── SellerProductPropertyAgentResource.php
│               ├── SellerProductPropertyValueAgentResource.php
│               ├── SellerProductSkuAgentResource.php
│               ├── SellerProductSkuAssociationAgentResource.php
│               ├── SellerProductSpecificationAgentResource.php
│               ├── SellerProductGalleryAgentResource.php
│               └── SellerProductSkuRegionPriceAgentResource.php
│
├── Models/
│   ├── SellerProduct.php
│   ├── SellerProductProxy.php
│   ├── SellerProductProperty.php
│   ├── SellerProductPropertyProxy.php
│   ├── SellerProductPropertyValue.php
│   ├── SellerProductPropertyValueProxy.php
│   ├── SellerProductSku.php
│   ├── SellerProductSkuProxy.php
│   ├── SellerProductSkuAssociation.php
│   ├── SellerProductSkuAssociationProxy.php
│   ├── SellerProductSpecification.php
│   ├── SellerProductSpecificationProxy.php
│   ├── SellerProductGallery.php
│   ├── SellerProductGalleryProxy.php
│   ├── SellerProductSkuRegionPrice.php
│   └── SellerProductSkuRegionPriceProxy.php
│
├── Observers/
│   └── SellerProductObserver.php
│
├── Policies/
│   └── SellerProductPolicy.php
│
├── Providers/
│   ├── AuthServiceProvider.php
│   ├── ModuleServiceProvider.php
│   └── SellerServiceProvider.php
│
├── Repositories/
│   ├── SellerProductRepository.php
│   ├── SellerProductPropertyRepository.php
│   ├── SellerProductSkuRepository.php
│   ├── SellerProductSpecificationRepository.php
│   ├── SellerProductGalleryRepository.php
│   └── SellerProductSkuRegionPriceRepository.php
│
├── Services/
│   ├── SellerProductService.php
│   ├── SellerProductPropertyService.php
│   ├── SellerProductSkuService.php
│   ├── SellerProductSpecificationService.php
│   ├── SellerProductGalleryService.php
│   ├── SellerProductEtagService.php
│   ├── SellerProductSkuRegionPriceService.php
│   ├── JsonPatchProcessor.php
│   └── CategorySchemaValidator.php
│
├── Resources/
│   └── manifest.php
│
└── Validators/
    ├── ShippingCategoryIdExists.php
    └── SellerProductIdExists.php
```
