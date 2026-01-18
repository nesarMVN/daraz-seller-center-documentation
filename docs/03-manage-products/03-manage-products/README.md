# Manage Products Documentation

## Overview

The **Manage Products** page is the central hub for viewing, organizing, filtering, and managing all products in the Daraz Seller Center. Sellers can track product status, edit listings, perform bulk operations, and monitor product performance from this interface.

**URL:** `https://sellercenter.daraz.com.bd/apps/product/list`

**Navigation Path:** Products > Manage Products

---

## Page Layout

### Header Section

The Manage Products header displays:

| Element | Description |
|---------|-------------|
| **Page Title** | "Manage Products" - appears at the top of the page |
| **Breadcrumb** | Home > Manage Products |
| **Product Data Button** | Links to product performance analytics |
| **Bulk Manage Dropdown** | Access to bulk operations menu |
| **New Product Button** | Quick access to add new products |

### Information Alert

A blue information alert appears below the header with helpful resources:
- Welcome message with "Learn More" link
- - Size Chart Tool information with "Learn how to use" link
  - - To-do list reminder if products are not yet visible to buyers
   
    - ---

    ## Product Overview Section

    The Product Overview section displays key metrics about your product inventory.

    ### Product Limitation

    Shows your product count against the maximum allowed:
    - **Progress Bar:** Visual representation of usage
    - - **Count Display:** Format: "- / 1,000" (current count / maximum allowed)
      - - **Show Details/Hide Button:** Toggle to expand/collapse detailed view
       
        - ---

        ## Product Status Tabs

        The Manage Products page organizes products into seven status tabs:

        ### 1. All Tab
        - **Purpose:** View all products regardless of status
        - - **Content:** Combined list of all products from all statuses
          - - **Use Case:** Get a complete overview of your entire product catalog
           
            - ### 2. Active Tab
            - - **Purpose:** View products currently live and visible to buyers
              - - **Content:** Products that are published and available for purchase
                - - **Indicators:** Shows products ready for sale
                  - - **Use Case:** Monitor your active inventory
                   
                    - ### 3. Inactive Tab
                    - - **Purpose:** View products that are not currently visible to buyers
                      - - **Content:** Products that have been deactivated or taken offline
                        - - **Filter Available:** "Out Of Stock" checkbox filter
                          - - **Use Case:** Manage products temporarily removed from sale
                           
                            - ### 4. Draft Tab
                            - - **Purpose:** View unfinished product listings
                              - - **Content:** Products that were started but not completed
                                - - **Badge:** Shows count of draft products (e.g., "1")
                                  - - **Actions Available:** Edit or Delete each draft
                                    - - **Status Message:** "Product won't be seen by buyer" with last updated timestamp
                                      - - **Use Case:** Complete or remove incomplete product listings
                                       
                                        - ### 5. Pending QC Tab
                                        - - **Purpose:** View products awaiting quality control review
                                          - - **Content:** Newly submitted products under Daraz review
                                            - - **Use Case:** Track products pending approval
                                             
                                              - ### 6. Violation Tab
                                              - - **Purpose:** View products flagged for policy violations
                                                - - **Content:** Products that violate Daraz's listing policies
                                                  - - **Use Case:** Address and fix policy violations
                                                   
                                                    - ### 7. Deleted Tab
                                                    - - **Purpose:** View products that have been deleted
                                                      - - **Content:** Permanently removed product listings
                                                        - - **Use Case:** Review deleted products
                                                         
                                                          - ---

                                                          ## Search and Filter Functionality

                                                          ### Search Section

                                                          The search and filter bar provides powerful product discovery tools:

                                                          | Element | Type | Description |
                                                          |---------|------|-------------|
                                                          | **Search Type Dropdown** | Combobox | Choose search criteria (default: "Product Name") |
                                                          | **Search Input** | Text Field | Enter search keywords (placeholder: "Please Input") |
                                                          | **Search Button** | Button | Execute the search query |
                                                          | **Select Category** | Dropdown | Filter products by category |
                                                          | **Sort By** | Dropdown | Sort results by various criteria |

                                                          ### Filter Options

                                                          **Out Of Stock Filter:**
                                                          - Checkbox filter available on Inactive tab
                                                          - - Quickly identify products with no inventory
                                                            - - Helps manage stock replenishment
                                                             
                                                              - **Reset Function:**
                                                              - - Some tabs include a "Reset" button
                                                                - - Clears all applied filters and searches
                                                                  - - Returns to default view
                                                                   
                                                                    - ---

                                                                    ## Bulk Operations

                                                                    The **Bulk Manage** dropdown menu provides access to powerful batch editing tools:

                                                                    ### Available Bulk Operations

                                                                    | Operation | Description | Badge |
                                                                    |-----------|-------------|-------|
                                                                    | **Bulk Add** | Upload multiple products at once via spreadsheet |  |
                                                                    | **Bulk Edit** | Edit multiple product fields simultaneously |  |
                                                                    | **Manage Image** | AI-powered bulk image management | AI-Powered |
                                                                    | **Manage Size Chart** | Add or update size charts for fashion items |  |

                                                                    ### Using Bulk Operations

                                                                    1. Click the "Bulk Manage" dropdown button
                                                                    2. 2. Select the desired operation
                                                                       3. 3. Follow the specific workflow for that operation
                                                                          4. 4. Bulk operations typically involve:
                                                                             5.    - Downloading template files
                                                                                   -    - Filling in product data
                                                                                        -    - Uploading completed files
                                                                                             -    - Reviewing and confirming changes
                                                                                              
                                                                                                  - ---

                                                                                                  ## Product List View

                                                                                                  ### Product Cards

                                                                                                  Each product in the list displays:

                                                                                                  - **Checkbox:** Select product for bulk actions
                                                                                                  - - **Product Image:** Thumbnail of main product photo
                                                                                                    - - **Product Name:** Title in both English and local language
                                                                                                      - - **Status Badge:** Color-coded status indicator
                                                                                                        - - **Status Message:** Detailed status information with timestamps
                                                                                                          - - **Action Buttons:**
                                                                                                            -   - **Edit:** Modify product details
                                                                                                                -   - **Delete:** Remove product (for drafts)
                                                                                                                    -   - **Other actions:** Vary based on product status
                                                                                                                     
                                                                                                                        - ### Selection Tools
                                                                                                                     
                                                                                                                        - **Bulk Selection:**
                                                                                                                        - - **Header Checkbox:** Select/deselect all products on current page
                                                                                                                          - - **Product Count:** Display of selected items (e.g., "0 products selected")
                                                                                                                            - - **Batch Actions:** Available when products are selected (e.g., "Delete" button)
                                                                                                                             
                                                                                                                              - ---
                                                                                                                              
                                                                                                                              ## Empty State Messages
                                                                                                                              
                                                                                                                              When no products match the current filter or status, the page displays:
                                                                                                                              
                                                                                                                              **Visual:**
                                                                                                                              - Illustration of an empty box
                                                                                                                              - - Clean, centered layout
                                                                                                                               
                                                                                                                                - **Message:**
                                                                                                                                - - Primary: "No product under this status or filter"
                                                                                                                                  - - Secondary: "Please check other product status or use other filter."
                                                                                                                                   
                                                                                                                                    - **Purpose:** Guide sellers to adjust their view or add products
                                                                                                                                   
                                                                                                                                    - ---
                                                                                                                                    
                                                                                                                                    ## Action Buttons
                                                                                                                                    
                                                                                                                                    ### Primary Actions
                                                                                                                                    
                                                                                                                                    **Product Data Button:**
                                                                                                                                    - **Style:** Orange outlined button
                                                                                                                                    - - **Function:** Navigate to product performance analytics
                                                                                                                                      - - **Link:** `/ba/product/performance?dateType=recent30`
                                                                                                                                       
                                                                                                                                        - **New Product Button:**
                                                                                                                                        - - **Style:** Orange filled button with "+" icon
                                                                                                                                          - - **Function:** Quick access to add product page
                                                                                                                                            - - **Link:** `/apps/product/publish`
                                                                                                                                             
                                                                                                                                              - ---
                                                                                                                                              
                                                                                                                                              ## Workflow Examples
                                                                                                                                              
                                                                                                                                              ### View All Draft Products
                                                                                                                                              
                                                                                                                                              1. Navigate to **Products > Manage Products**
                                                                                                                                              2. 2. Click the **Draft** tab
                                                                                                                                                 3. 3. Review list of unfinished products
                                                                                                                                                    4. 4. Click **Edit** to complete a draft
                                                                                                                                                       5. 5. Or click **Delete** to remove unwanted drafts
                                                                                                                                                         
                                                                                                                                                          6. ### Filter Inactive Out-of-Stock Products
                                                                                                                                                         
                                                                                                                                                          7. 1. Navigate to **Products > Manage Products**
                                                                                                                                                             2. 2. Click the **Inactive** tab
                                                                                                                                                                3. 3. Check the **Out Of Stock** filter checkbox
                                                                                                                                                                   4. 4. Review products needing inventory replenishment
                                                                                                                                                                      5. 5. Update stock levels or reactivate products
                                                                                                                                                                        
                                                                                                                                                                         6. ### Bulk Edit Multiple Products
                                                                                                                                                                        
                                                                                                                                                                         7. 1. Navigate to **Products > Manage Products**
                                                                                                                                                                            2. 2. Select a status tab with products
                                                                                                                                                                               3. 3. Check boxes next to products to edit
                                                                                                                                                                                  4. 4. Click **Bulk Manage** dropdown
                                                                                                                                                                                     5. 5. Select **Bulk Edit**
                                                                                                                                                                                        6. 6. Follow bulk edit workflow
                                                                                                                                                                                           7. 7. Review and confirm changes
                                                                                                                                                                                             
                                                                                                                                                                                              8. ### Search for Specific Product
                                                                                                                                                                                             
                                                                                                                                                                                              9. 1. Navigate to **Products > Manage Products**
                                                                                                                                                                                                 2. 2. Ensure **Product Name** is selected in search dropdown
                                                                                                                                                                                                    3. 3. Type product name in search field
                                                                                                                                                                                                       4. 4. Click **Search** button
                                                                                                                                                                                                          5. 5. Review filtered results
                                                                                                                                                                                                             6. 6. Click **Reset** to clear search
                                                                                                                                                                                                               
                                                                                                                                                                                                                7. ---
                                                                                                                                                                                                               
                                                                                                                                                                                                                8. ## Product Overview Details
                                                                                                                                                                                                               
                                                                                                                                                                                                                9. When **Show Details** is clicked, the Product Overview expands to show:
                                                                                                                                                                                                               
                                                                                                                                                                                                                10. ### Product Limitation Section
                                                                                                                                                                                                               
                                                                                                                                                                                                                11. **Display Format:**
                                                                                                                                                                                                                12. - Section title: "Product Limitation"
                                                                                                                                                                                                                    - - Progress bar showing usage percentage
                                                                                                                                                                                                                      - - Numerical display: "- / 1,000"
                                                                                                                                                                                                                        - - Information icon for additional help
                                                                                                                                                                                                                         
                                                                                                                                                                                                                          - **Purpose:**
                                                                                                                                                                                                                          - - Monitor how many products you can still add
                                                                                                                                                                                                                            - - Prevent hitting product limits
                                                                                                                                                                                                                              - - Plan inventory growth
                                                                                                                                                                                                                               
                                                                                                                                                                                                                                - ---
                                                                                                                                                                                                                                
                                                                                                                                                                                                                                ## Tips and Best Practices
                                                                                                                                                                                                                                
                                                                                                                                                                                                                                ### Product Management
                                                                                                                                                                                                                                
                                                                                                                                                                                                                                1. **Regular Review:** Check each status tab weekly to identify issues
                                                                                                                                                                                                                                2. 2. **Complete Drafts:** Finish or delete draft products to keep workspace clean
                                                                                                                                                                                                                                   3. 3. **Monitor Violations:** Address violation tab products immediately to restore sales
                                                                                                                                                                                                                                      4. 4. **Stock Management:** Use Out Of Stock filter regularly to maintain inventory
                                                                                                                                                                                                                                        
                                                                                                                                                                                                                                         5. ### Search and Organization
                                                                                                                                                                                                                                        
                                                                                                                                                                                                                                         6. 1. **Use Categories:** Filter by category for easier navigation
                                                                                                                                                                                                                                            2. 2. **Descriptive Names:** Use clear product names for easier searching
                                                                                                                                                                                                                                               3. 3. **Bulk Operations:** Leverage bulk tools for efficiency on large catalogs
                                                                                                                                                                                                                                                  4. 4. **Sort Strategically:** Use Sort By to prioritize products needing attention
                                                                                                                                                                                                                                                    
                                                                                                                                                                                                                                                     5. ### Performance Tracking
                                                                                                                                                                                                                                                    
                                                                                                                                                                                                                                                     6. 1. **Product Data:** Regularly check Product Data analytics
                                                                                                                                                                                                                                                        2. 2. **Active Products:** Keep Active tab optimized for maximum visibility
                                                                                                                                                                                                                                                           3. 3. **Quality Control:** Monitor Pending QC tab for approval status
                                                                                                                                                                                                                                                              4. 4. **Inactive Review:** Periodically review Inactive products for reactivation opportunities
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 5. ---
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 6. ## Screenshots Reference
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 7. ### 1. Manage Products Main Page (Active Tab)
                                                                                                                                                                                                                                                                 8. ![Manage Products Overview](images/screenshot-01-manage-products-overview.png)
                                                                                                                                                                                                                                                                 9. *Main Manage Products page showing the Active tab with search filters and action buttons*
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 10. ---
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 11. ### 2. All Products Tab
                                                                                                                                                                                                                                                                 12. ![All Products Tab](images/screenshot-02-all-products-tab.png)
                                                                                                                                                                                                                                                                 13. *All tab view showing combined product list and filter options*
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 14. ---
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 15. ### 3. Draft Products Tab
                                                                                                                                                                                                                                                                 16. ![Draft Products Tab](images/screenshot-03-draft-tab-with-product.png)
                                                                                                                                                                                                                                                                 17. *Draft tab showing an incomplete product listing with Edit and Delete options*
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 18. ---
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 19. ### 4. Bulk Manage Dropdown Menu
                                                                                                                                                                                                                                                                 20. ![Bulk Manage Menu](images/screenshot-04-bulk-manage-dropdown.png)
                                                                                                                                                                                                                                                                 21. *Bulk Manage dropdown displaying available bulk operation options including AI-Powered image management*
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 22. ---
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 23. ### 5. Search Filter Dropdown
                                                                                                                                                                                                                                                                 24. ![Search Filter Options](images/screenshot-05-search-filter-dropdown.png)
                                                                                                                                                                                                                                                                 25. *Product Name search filter dropdown showing available search criteria*
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 26. ---
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 27. ### 6. Product Overview Expanded
                                                                                                                                                                                                                                                                 28. ![Product Overview Details](images/screenshot-06-product-overview-expanded.png)
                                                                                                                                                                                                                                                                 29. *Expanded Product Overview section showing Product Limitation progress bar and details*
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 30. ---
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 31. ### 7. Inactive Tab with Filter
                                                                                                                                                                                                                                                                 32. ![Inactive Tab](images/screenshot-07-inactive-tab-out-of-stock.png)
                                                                                                                                                                                                                                                                 33. *Inactive tab with Out Of Stock filter checkbox and empty state message*
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 34. ---
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 35. ## Related Documentation
                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                 36. - [Add Product Documentation](../01-add-product/README.md)
                                                                                                                                                                                                                                                                     - - [Media Center Documentation](../02-media-center/README.md)
