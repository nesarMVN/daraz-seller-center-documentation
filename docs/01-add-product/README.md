# Add Product - Complete Documentation

> **Module:** Products
> > **Page:** Add Product
> > > **URL:** `https://sellercenter.daraz.com.bd/apps/product/publish`
> > >
> > > ---
> > >
> > > ## Overview
> > >
> > > The Add Product page allows sellers to create new product listings on Daraz. This documentation covers all sections and features of the product creation form.
> > >
> > > ## Table of Contents
> > >
> > > 1. [Basic Information](#1-basic-information)
> > > 2. 2. [Product Images](#2-product-images)
> > >    3. 3. [Video Section](#3-video-section)
> > >       4. 4. [Product Specification](#4-product-specification)
> > >          5. 5. [Price, Stock & Variants](#5-price-stock--variants)
> > >             6. 6. [Product Description](#6-product-description)
> > >                7. 7. [Shipping & Warranty](#7-shipping--warranty)
> > >                  
> > >                   8. ---
> > >                  
> > >                   9. ## 1. Basic Information
> > >                  
> > >                   10. ### Screenshot: Empty Form
> > >
> > > ![Empty Form](./images/screenshot-02-add-product-empty-form.png)
> > >
> > > | Field | Type | Required | Description |
> > > |-------|------|----------|-------------|
> > > | Product Name | Text Input | Yes (*) | Maximum 255 characters. Format: Product Name + Type + Key Features |
> > > | Category | Selector | Yes (*) | Choose from predefined category tree |
> > >
> > > ### AI Category Suggestions
> > >
> > > ![AI Suggestions](./images/screenshot-03-product-name-with-ai-suggestions.png)
> > >
> > > When you enter a product name, AI automatically suggests relevant categories based on keywords.
> > >
> > > **Example:** For "Samsung Galaxy A54 5G Smartphone 128GB Black":
> > > - Mobiles & Tablets > Smart Phones ✓
> > > - - Mobiles & Tablets > Tablets
> > >   - - Mobiles & Tablets > Mobile Phone Accessories
> > >    
> > >     - ---
> > >
> > > ## 2. Product Images
> > >
> > > ### Upload Options
> > >
> > > ![Upload Options](./images/screenshot-05-product-images-upload-options.png)
> > >
> > > | Option | Description |
> > > |--------|-------------|
> > > | Upload | Upload from local computer |
> > > | Media Center | Select from your Media Center library |
> > >
> > > ### Requirements
> > >
> > > - **Minimum:** 3 images required
> > > - - **Maximum:** 8 images allowed
> > >   - - **Format:** JPG, PNG, JPEG
> > >     - - **Size:** Less than 6MB per image
> > >       - - **Background:** White or light-colored preferred
> > >        
> > >         - ### Media Center Modal
> > >        
> > >         - ![Media Center](./images/screenshot-06-media-center-modal.png)
> > >        
> > >         - ### Image Guidelines
> > >
> > > ![Guidelines](./images/screenshot-08-product-images-example-guidelines.png)
> > >
> > > **Do's:**
> > > - Clean white background
> > > - - Product clearly visible
> > >   - - Professional lighting
> > >     - - High resolution
> > >      
> > >       - **Don'ts:**
> > >       - - Cluttered background
> > >         - - Watermarks or logos
> > >           - - Text overlays
> > >             - - Poor lighting
> > >              
> > >               - ---
> > >
> > > ## 3. Video Section
> > >
> > > ![Video Options](./images/screenshot-09-video-youtube-link-option.png)
> > >
> > > | Option | Description |
> > > |--------|-------------|
> > > | Upload | Direct video upload (max 100MB) |
> > > | YouTube Link | Embed existing YouTube video |
> > >
> > > **Supported formats:** wmv, avi, mpg, mpeg, 3gp, mov, mp4, flv, f4v, m4v, m2t, mts, rmvb, vob, mkv
> > >
> > > ---
> > >
> > > ## 4. Product Specification
> > >
> > > ### Brand Selection
> > >
> > > ![Brand Search](./images/screenshot-10-product-specification-brand-search.png)
> > >
> > > - Searchable dropdown
> > > - - Fill Rate contribution: 13%
> > >  
> > >   - ### Screen Size
> > >  
> > >   - ![Screen Size](./images/screenshot-12-screen-size-dropdown.png)
> > >  
> > >   - Options: Less than 5 Inch, 5 Inch, 5.1-5.4 Inch, 5.5 Inch, 5.6-5.9 Inch, 6 Inch and Above
> > >
> > >   - ### Additional Specifications
> > >
> > >   - ![More Fields](./images/screenshot-14-show-more-additional-fields.png)
> > >
> > >   - | Field | Type | Fill Rate Impact |
> > >   - |-------|------|-----------------|
> > >   - | Network Connections | Dropdown | Variable |
> > >   - | Operating System | Dropdown | Variable |
> > >   - | Features | Multi-select | Variable |
> > >   - | RAM Memory | Dropdown | Variable |
> > >   - | Storage Capacity | Dropdown | Variable |
> > >   - | Battery Capacity | Text Input | Variable |
> > >   - | Screen Type | Dropdown | Variable |
> > >   - | Camera (rear) | Text Input | Variable |
> > >
> > >   - ---
> > >
> > > ## 5. Price, Stock & Variants
> > >
> > > ### Variant Configuration
> > >
> > > ![Variants Initial](./images/screenshot-16-price-stock-variants-initial.png)
> > >
> > > **Available variant types:**
> > > - Color Family
> > > - - Storage Capacity
> > >   - - Size
> > >     - - Bundle
> > >      
> > >       - ### Color Selection
> > >      
> > >       - ![Color Family](./images/screenshot-17-variant1-color-family-dropdown.png)
> > >      
> > >       - ### Storage Options
> > >
> > > ![Storage](./images/screenshot-19-variant2-storage-capacity.png)
> > >
> > > Options: 8GB, 16GB, 32GB, 64GB, 128GB, 256GB, 512GB, 1TB, 2TB
> > >
> > > ### Price & Stock Table
> > >
> > > ![Price Table](./images/screenshot-20-variants-configured-price-stock-table.png)
> > >
> > > | Column | Type | Required |
> > > |--------|------|----------|
> > > | Variant | Display | - |
> > > | Price (৳) | Number | Yes |
> > > | Special Price (৳) | Number | Yes |
> > > | Seller SKU | Text | Yes |
> > > | Free Items | Text | No |
> > > | Quantity | Number | Yes |
> > > | SellerSKU Image | Upload | No |
> > >
> > > ---
> > >
> > > ## 6. Product Description
> > >
> > > ### Rich Text Editor
> > >
> > > ![Editor](./images/screenshot-21-product-description-main-editor.png)
> > >
> > > **Toolbar features:** Bold, Italic, Underline, Font Size, Text Color, Alignment, Lists, Image, Link, Table
> > >
> > > ### Advanced Mode
> > >
> > > ![Advanced Mode](./images/screenshot-23-advanced-mode-component-tab.png)
> > >
> > > **Components:**
> > > - Image layouts (1-4 slots)
> > > - - Text & Image combinations
> > >   - - Free Design (drag-and-drop)
> > >    
> > >     - ### Pre-built Templates
> > >    
> > >     - ![Templates](./images/screenshot-25-lazada-design-templates.png)
> > >    
> > >     - | Template | Industry |
> > > |----------|----------|
> > > | EL Template | Electronics |
> > > | FMCG Template | Fast-Moving Consumer Goods |
> > > | Fashion Template | Fashion/Apparel |
> > >
> > > ---
> > >
> > > ## 7. Shipping & Warranty
> > >
> > > ### Package Information
> > >
> > > ![Shipping](./images/screenshot-26-shipping-warranty-basic.png)
> > >
> > > | Field | Type | Unit | Required |
> > > |-------|------|------|----------|
> > > | Package Weight | Number | kg | Yes |
> > > | Package Length | Number | cm | Yes |
> > > | Package Width | Number | cm | Yes |
> > > | Package Height | Number | cm | Yes |
> > >
> > > ### Dangerous Goods
> > >
> > > Options: None, Contains Battery, Flammable, Other
> > >
> > > ### Warranty Settings
> > >
> > > ![Warranty](./images/screenshot-27-shipping-warranty-full.png)
> > >
> > > | Field | Type | Options |
> > > |-------|------|---------|
> > > | Warranty Type | Dropdown | No Warranty, Brand, Seller, Local |
> > > | Warranty Duration | Dropdown | 1 month to 2+ years |
> > > | Warranty Policy | Text Area | Free text |
> > > | Return Policy | Text Input | Custom policy |
> > >
> > > ---
> > >
> > > ## Summary
> > >
> > > **Total Sections:** 7 main sections
> > > **Required Fields:** Product Name, Category, Images (min 3), Price, SKU, Quantity, Package Info
> > >
> > > ---
> > >
> > > *Last Updated: January 17, 2026*
