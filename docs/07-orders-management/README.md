# Orders Management Documentation

## Overview

**URL:** https://sellercenter.daraz.com.bd/apps/order/list

**Navigation:** Orders and Reviews → Orders

The Orders Management system is the central hub for processing, tracking, and managing all customer orders. It provides comprehensive tools for order fulfillment, from initial payment confirmation through shipping, delivery, and handling returns or cancellations. Sellers can monitor order statuses, manage fulfillment timelines (SLA), print shipping documents, and export order data.

## Page Layout

### Header Section

- **Page Title:** "Order Management"
- **Breadcrumb Navigation:** Homepage → Order Management
- **Main Heading:** "Order Management" displayed prominently

### Status Tabs

Eight horizontal tabs for filtering orders by status:

1. **All** - View all orders regardless of status
2. **Unpaid** - Orders awaiting customer payment
3. **To Ship** - Orders ready to be packed and shipped (Default view)
4. **Shipping** - Orders currently in transit
5. **Delivered** - Orders successfully delivered to customers
6. **Failed Delivery** - Orders with delivery issues
7. **Cancellation** - Cancelled orders
8. **Return Or Refund** - Orders with return/refund requests

---

## To Ship Tab (Primary Fulfillment View)

**Purpose:** Main workspace for processing orders ready for shipment.

![To Ship Main View](images/screenshot-01-to-ship-main.png)

*Order Management page showing "To Ship" tab selected, three sub-status filters (To Pack, To Arrange Shipment, To Handover), NEW Fulfillment SLA section with breach checkboxes, Print Status filters (AWB, Invoice, PickList), Order Number search, and empty data table with Product, Total Amount, Delivery, Status, Actions columns.*

### Sub-Status Filters

Three radio button options for refined order filtering:

1. **To Pack** - Orders needing to be packed
   - Initial state after payment confirmation
   - Seller needs to prepare items for shipping

2. **To Arrange Shipment** - Orders packed, awaiting courier pickup arrangement
   - Items packed and ready
   - Shipping label printed
   - Need to schedule logistics pickup

3. **To Handover** - Orders ready for physical handover to courier
   - All documentation complete
   - Awaiting courier collection
   - Link: "Go to Logistics Center to Overview Handover" (redirects to https://scheduling.daraz.com.bd/seller/dashboard)

### Fulfillment SLA Filters

**NEW Label:** Indicates new feature

Two checkboxes for SLA monitoring:

- **About to Breach SLA(0)** - Orders approaching fulfillment deadline
  - Shows count of orders close to SLA limit
  - Requires urgent attention
  - Icon tooltip provides SLA details

- **SLA Breached(0)** - Orders that missed fulfillment deadline
  - Indicates performance impact
  - May affect seller metrics
  - Requires immediate action

### Print Status Filters

Six checkbox options for document printing status:

1. **AWB Unprinted** - Air Waybill not yet printed
2. **AWB Printed** - Air Waybill printed and ready
3. **Invoice Unprinted** - Customer invoice not printed
4. **Invoice Printed** - Customer invoice printed
5. **PickList Unprinted** - Warehouse picklist not printed
6. **PickList Printed** - Warehouse picklist printed

### Search and Filter Options

#### Basic Search

- **Order Number:** Text input with edit and search icons
  - Search by specific order ID
  - Supports bulk search with comma-separated values

#### Expanded Filters (Click "More" to reveal)

**Order Date Filter:**
- Today
- Yesterday
- Last 7 days
- Last 30 days
- Custom (with Start Date and End Date pickers)

**Product Filters:**
- **Product Name:** Text search for products within orders
- **Seller SKU:** Search by internal product SKU

**Customer Filters:**
- **Customer Name:** Dropdown search for customer orders

**Delivery Filters:**
- **Delivery Type:** Dropdown (Standard, Express, etc.)
- **Order Type:** Dropdown filter

**Payment Filters:**
- **Payment Channel:** Dropdown (COD, Card, Bank Transfer, etc.)

**Fold Button:** Collapses expanded filters back to basic view

![To Ship Expanded Filters](images/screenshot-02-to-ship-filters.png)

*"More" button clicked revealing expanded filter options: Order Date buttons (Today, Yesterday, Last 7 days, Last 30 days, Custom with date range), Product Name search, Delivery Type dropdown, Seller SKU search, Customer Name search, Order Type dropdown, Payment Channel dropdown, and "Fold" button to collapse filters.*

### Bulk Actions

Located above order table:

1. **Pack & Print** - Batch process multiple orders
   - Mark orders as packed
   - Print shipping labels
   - Generate invoices
   - Accessible when orders selected

2. **Print PickList** - Generate warehouse picking lists
   - Batch print picking documents
   - Organize packing workflow
   - Available when orders selected

3. **Export** - Download order data
   - CSV/Excel format options
   - Filtered results export
   - Comprehensive order details

### Sort Options

**Sort By:** Dropdown with options
- **Oldest Order Created** (Default for To Ship tab)
- Newest Order Created
- Order Amount (High to Low)
- Order Amount (Low to High)

### Order Table

**Column Headers:**

1. **Checkbox Column** - Select individual or all orders
2. **Product** - Product image, title, SKU, quantity
3. **Total Amount** - Order value in BDT (৳)
4. **Delivery** - Shipping address and delivery type
5. **Status** - Current order status with color coding
6. **Actions** - Order-specific action buttons

**Empty State:**
- Message: "Empty data"
- Appears when no orders match current filters
- Gentle gray text on white background

### Pagination Controls

Bottom of page:

- **Previous/Next buttons** - Navigate between pages
- **Page numbers** - Direct page navigation (1, 2, 3... with current page highlighted in orange)
- **Items per page:** Dropdown (default: 20)
  - Options: 10, 20, 50, 100
- **Page indicator:** "Page 1, 1 - 0 of 0 items" format

---

## All Tab

**Purpose:** View complete order history across all statuses.

![All Orders Tab](images/screenshot-03-all-orders-tab.png)

*"All" tab selected showing different interface: Order Type filter buttons (All, Normal), Order Number and Tracking Number search fields, Order Date filters, "More" button, table with same columns, Sort By dropdown set to "Newest Order Created", and Export button.*

### Unique Features

**Order Type Filter:**
- **All** (Default, orange button)
- **Normal** (Standard orders)

**Search Options:**
- Order Number
- Tracking Number (with search icon)

**Date Filters:**
- Same as To Ship tab (Today, Yesterday, Last 7 days, Last 30 days, Custom)

**Sort Default:**
- **Newest Order Created** (opposite of To Ship tab)

**Table Structure:**
- Same columns as To Ship tab
- Shows orders from all status categories
- Status column displays current state

**Export Functionality:**
- Export button available
- No Pack & Print or Print PickList (status-independent actions only)

---

## Unpaid Tab

**Purpose:** Monitor orders awaiting payment confirmation.

### Key Features

- **Payment Pending Orders:** Customer initiated order but hasn't completed payment
- **Auto-Cancellation:** Orders auto-cancel after set period (typically 24 hours)
- **Manual Cancellation:** Option to cancel unpaid orders
- **Payment Method Display:** Shows intended payment channel

### Typical Actions

- View order details
- Cancel unpaid orders (if needed)
- Monitor payment completion
- Contact customer (if payment issues)

---

## Shipping Tab

**Purpose:** Track orders currently in transit with courier.

### Tracking Information

- **Tracking Number Display:** Courier tracking ID visible
- **Shipping Status Updates:** Real-time status from logistics provider
- **Estimated Delivery Date:** Expected delivery timeline
- **Courier Information:** Logistics company handling delivery

### Monitoring Features

- Filter by courier/logistics provider
- Search by tracking number
- View delivery route (if available)
- Update tracking information

---

## Delivered Tab

**Purpose:** View completed successful deliveries.

### Order Completion

- **Delivery Confirmation:** Customer received order
- **Proof of Delivery:** Signature/OTP confirmation
- **Settlement Timeline:** Payment settlement schedule visible
- **Review Opportunities:** Customers can leave reviews

### Post-Delivery Actions

- View delivery proof
- Monitor for review submissions
- Handle any post-delivery inquiries
- Export delivery reports

---

## Failed Delivery Tab

**Purpose:** Manage orders with delivery issues.

### Common Failure Reasons

- Customer unavailable
- Incorrect address
- Refused delivery
- Delivery area inaccessible
- Customer rescheduled

### Resolution Actions

- **Reattempt Delivery:** Schedule new delivery attempt
- **Contact Customer:** Update delivery details
- **Return to Warehouse:** Initiate return process
- **Cancel Order:** Process refund if undeliverable

### Retry Management

- Track delivery attempt count
- View failure reasons
- Update customer contact information
- Set new delivery schedule

---

## Cancellation Tab

**Purpose:** Manage cancelled orders (seller or customer initiated).

### Cancellation Types

**Customer Cancelled:**
- Buyer requested cancellation before shipment
- Reasons: Changed mind, found better price, wrong item, etc.
- Refund processing required

**Seller Cancelled:**
- Out of stock
- Pricing error
- Cannot fulfill order
- Buyer request compliance

### Cancellation Workflow

1. View cancellation reason
2. Process refund (if payment received)
3. Restore inventory
4. Update order status
5. Monitor refund completion

### Impact on Metrics

- Cancellation rate tracked
- Affects seller performance score
- High rates may trigger review
- Platform monitoring for patterns

---

## Return Or Refund Tab

**Purpose:** Handle product returns and refund requests.

### Return Types

**Return to Daraz Warehouse:**
- Customer ships item back to Daraz facility
- Quality check by Daraz team
- Refund processed after inspection
- Item restocked if conditions met

**Refund Only:**
- Customer keeps product
- Refund issued without return
- Used for: Damaged goods, wrong item, defective products
- Seller bears cost without item recovery

### Return Status Workflow

**Return Initiated:**
- Customer requested return
- Awaiting approval
- Review return reason
- Accept or reject request

**Return in Progress:**
- Return approved
- Item being shipped back
- Tracking active
- Awaiting receipt at warehouse

**QC in Progress:**
- Item received at warehouse
- Quality check underway
- Inspecting condition
- Verifying return reason

**Returned:**
- QC complete
- Item accepted
- Refund processed
- Order closed

**Scrapped:**
- Item damaged/unusable
- Cannot be restocked
- Disposed/recycled
- Refund still processed

**Cancelled:**
- Return request cancelled
- Customer kept item
- No refund processed
- Order remains delivered

### Return Management Features

**Search Options:**
- Order ID dropdown with search
- Customer Phone with search

**Date Filters:**
- Today, Yesterday, Last 7 days, Last 30 days, Custom

**Action Buttons:**
- Export returns data
- View history of returns

---

## Left Sidebar Navigation

### Orders and Reviews Menu

**Sub-menu items:**

1. **Orders** (Current page)
   - Icon: Box/package symbol
   - Main order management hub

2. **Return Orders** (Separate detailed view)
   - URL: https://sellercenter.daraz.com.bd/apps/order/reverse
   - Dedicated return management interface
   - More detailed than Return Or Refund tab

3. **Reviews**
   - Customer review management
   - Respond to reviews
   - Monitor ratings

---

## Return Orders Page (Dedicated View)

**URL:** https://sellercenter.daraz.com.bd/apps/order/reverse

![Return Orders Main](images/screenshot-04-return-orders-main.png)

*Separate Return Orders page with "Return to Daraz Warehouse" and "Refund Only" toggle buttons, green feedback banner, seven status tabs (All, Return Initiated, Return in Progress, QC in Progress, Returned, Scrapped, Cancelled with info icons), date filter buttons, Order ID and Customer Phone search dropdowns, Export and View history buttons, and decorative empty state illustration.*

### Top Options

Two toggle buttons:

1. **Return to Daraz Warehouse** (Default)
   - Full return process
   - Item shipped to warehouse
   - QC inspection required

2. **Refund Only**
   - Refund without return
   - Customer keeps item
   - Faster resolution

### Feedback Banner

Green information banner:
- "Tell us more about your Return management experience with Lazada. Give your feedback here!"
- Link to feedback form
- Dismissible notification

### Status Tabs

Seven status tabs for detailed return tracking:

1. **All** - All return requests
2. **Return Initiated** - New return requests
3. **Return in Progress** - Items being shipped back
4. **QC in Progress** - Quality check underway
5. **Returned** (with info icon) - Completed returns
6. **Scrapped** (with info icon) - Damaged items
7. **Cancelled** (with info icon) - Cancelled returns

**Info Icons:** Hover for detailed status explanations

### Search and Filter

**Date Range:**
- Today, Yesterday, Last 7 days, Last 30 days, Custom

**Search Fields:**
- **Order ID:** Dropdown with text input
- **Customer Phone:** Dropdown with text input

### Action Buttons

- **Export** - Download return data
- **View history** - Historical return records

### Empty State

Decorative illustration:
- Magnifying glass icon
- Pastel gradient background (pink, blue, purple)
- Message: "Page 1, 1 - 0 of 0 items"
- Friendly, approachable design

---

## Key Workflows

### Workflow 1: Processing New Orders (To Pack → To Handover)

**Step 1: Check New Orders**
1. Navigate to Orders → To Ship tab
2. Select "To Pack" sub-status
3. Review orders needing fulfillment
4. Check SLA timers - prioritize "About to Breach SLA"

**Step 2: Print Documents**
1. Select orders to process (checkbox)
2. Click "Pack & Print" button
3. System generates:
   - Air Waybill (AWB) - Shipping label
   - Invoice - Customer bill
   - PickList - Warehouse picking document
4. Print all documents

**Step 3: Pack Orders**
1. Use PickList to gather items from inventory
2. Verify product details match order
3. Pack securely with appropriate materials
4. Attach AWB shipping label
5. Include printed invoice
6. Mark items as packed in system

**Step 4: Arrange Shipment**
1. Orders automatically move to "To Arrange Shipment"
2. Schedule courier pickup
3. Use Logistics Center link or contact courier directly
4. Confirm pickup time and location

**Step 5: Handover to Courier**
1. Orders move to "To Handover"
2. Prepare orders for courier arrival
3. Hand over packages with documentation
4. Get courier receipt/confirmation
5. Update system with handover completion

**Step 6: Monitor Shipping**
1. Orders automatically move to "Shipping" tab
2. Track delivery progress via tracking number
3. Monitor for "Failed Delivery" status
4. Address any delivery issues promptly

**Step 7: Delivery Confirmation**
1. Order moves to "Delivered" upon successful delivery
2. Customer confirms receipt
3. Payment settlement initiated
4. Monitor for customer reviews

### Workflow 2: Handling Customer Returns

**Step 1: Return Request Received**
1. Customer initiates return request
2. Notification appears in "Return Or Refund" tab
3. Also visible in dedicated "Return Orders" page
4. Review return reason provided by customer

**Step 2: Evaluate Return Request**
1. Navigate to Return Orders page
2. Click on "Return Initiated" tab
3. Review order details:
   - Original order information
   - Product condition claims
   - Customer reason for return
   - Time since delivery
4. Check return policy compliance

**Step 3: Approve or Reject**
1. If valid: Approve return
   - Customer receives return shipping label
   - Instructions sent for return process
   - Status moves to "Return in Progress"
2. If invalid: Reject return
   - Provide clear rejection reason
   - Offer alternative solutions if applicable
   - Document decision

**Step 4: Track Return Shipment**
1. Monitor "Return in Progress" tab
2. Track item shipping back to warehouse
3. Estimated arrival date displayed
4. Update customer on progress

**Step 5: Quality Check**
1. Item arrives at Daraz warehouse
2. Status changes to "QC in Progress"
3. Warehouse team inspects:
   - Product condition
   - Completeness (accessories included)
   - Damage assessment
   - Authenticity verification

**Step 6: QC Decision**
1. **If Approved:**
   - Status: "Returned"
   - Refund processed automatically
   - Item returned to inventory (if resellable)
   - Customer notified

2. **If Damaged:**
   - Status: "Scrapped"
   - Item disposed
   - Refund still processed (customer not at fault if seller sent defective)
   - Seller may dispute if damage occurred post-delivery

3. **If Fraudulent:**
   - Status: "Cancelled"
   - Return rejected
   - No refund issued
   - Customer keeps item or pays return costs

**Step 7: Resolution & Monitoring**
1. Monitor refund completion
2. Update inventory if item returned
3. Analyze return reasons for patterns
4. Improve product listings/descriptions to reduce returns
5. Track return rate impact on seller metrics

### Workflow 3: Handling Failed Deliveries

**Step 1: Identify Failed Delivery**
1. Order appears in "Failed Delivery" tab
2. Review failure reason from courier
3. Check delivery attempt count

**Step 2: Analyze Failure Reason**

**Common Scenarios:**

**Customer Unavailable:**
- Customer not home during delivery
- Phone unreachable
- No response to courier attempts

**Action:**
- Contact customer via Daraz messaging
- Reschedule delivery
- Confirm availability window
- Arrange second delivery attempt

**Incorrect Address:**
- Address incomplete or wrong
- Location not found by courier
- Building/apartment details missing

**Action:**
- Request corrected address from customer
- Update delivery details in system
- Verify new address with customer
- Reattempt delivery with updated info

**Customer Refused:**
- Customer rejected delivery at door
- No longer wants order
- Found alternative/changed mind

**Action:**
- Initiate return process
- Process cancellation
- Arrange refund
- Item returns to warehouse

**Delivery Area Issues:**
- Remote location
- Area inaccessible by standard courier
- Security restrictions

**Action:**
- Arrange alternative delivery method
- Use different courier if available
- Customer pickup at designated location
- Negotiate solution with customer

**Step 3: Reattempt or Cancel**

**If Reattempt:**
1. Update customer contact information
2. Schedule new delivery date
3. Confirm customer availability
4. Assign to courier
5. Monitor second attempt

**If Cancel:**
1. Contact customer to confirm cancellation
2. Initiate return to warehouse
3. Process refund (if prepaid)
4. Close order
5. Restock inventory

**Step 4: Monitor Resolution**
1. Track reattempt delivery status
2. Follow up with customer if needed
3. Escalate to support if multiple failures
4. Document resolution for reference

### Workflow 4: Bulk Order Processing

**Step 1: Filter Orders for Batch Processing**
1. Navigate to "To Ship" tab
2. Select "To Pack" sub-status
3. Apply filters:
   - Date range: Select specific period
   - Product Name: If processing specific products
   - Delivery Type: Group by delivery method
4. Click "More" to access all filters

**Step 2: Prioritize by SLA**
1. Check "About to Breach SLA" checkbox
2. Sort by "Oldest Order Created"
3. Process urgent orders first
4. Note SLA deadlines

**Step 3: Batch Select Orders**
1. Click checkbox in table header to select all visible
2. Or manually select specific orders
3. Review selected count
4. Ensure manageable batch size (don't exceed packing capacity)

**Step 4: Batch Print Documents**
1. Click "Pack & Print" button
2. System generates combined document set:
   - All AWB labels in sequence
   - All invoices together
   - Consolidated PickList
3. Print all documents at once
4. Organize prints by order number

**Step 5: Efficient Packing Process**
1. Use consolidated PickList to gather all items
2. Group items by warehouse location for efficiency
3. Pack orders in sequence
4. Match products to order numbers
5. Quality check each package
6. Attach labels and invoices

**Step 6: Batch Update Status**
1. Mark all packed orders complete
2. System automatically moves to "To Arrange Shipment"
3. Schedule single courier pickup for all
4. Prepare for bulk handover

**Step 7: Export for Records**
1. Select processed orders
2. Click "Export" button
3. Download order details
4. Save for accounting/inventory records
5. Archive for reference

---

## Tips for Success

### Order Fulfillment Excellence

**SLA Management:**
- Check SLA status multiple times daily
- Prioritize "About to Breach" orders
- Pack orders within 24 hours of payment
- Build buffer time for courier delays
- Understand SLA impacts on seller rating

**Document Organization:**
- Print documents in batches for efficiency
- Keep AWB labels organized by order number
- File invoices for accounting
- Maintain PickList copies for inventory tracking
- Use color coding for priority orders

**Packing Best Practices:**
- Double-check product matches order
- Use appropriate packaging materials
- Protect fragile items adequately
- Include all accessories and manuals
- Secure labels firmly
- Add "Fragile" stickers when needed

**Communication:**
- Update customers on shipment status
- Provide tracking numbers promptly
- Respond to delivery inquiries quickly
- Proactive messaging reduces failed deliveries
- Use Daraz messaging system for records

### Return Management Optimization

**Reduce Return Rates:**
- Accurate product descriptions prevent wrong expectations
- High-quality photos show true product condition
- List all specifications clearly
- Mention common issues/limitations
- Size charts for apparel/footwear
- Respond to pre-purchase questions

**Efficient Return Processing:**
- Review return requests within 24 hours
- Approve valid returns quickly
- Clear communication of return process
- Track QC progress
- Analyze return patterns
- Address recurring issues

**Return Reason Analysis:**
- Track most common return reasons
- Identify problematic products
- Improve listings for frequent returns
- Consider discontinuing items with high return rates
- Use data to enhance product selection

### Performance Monitoring

**Key Metrics to Track:**
- Order fulfillment time (hours from payment to handover)
- SLA breach rate
- Failed delivery percentage
- Return rate by product/category
- Customer satisfaction scores
- Response time to inquiries

**Daily Routine:**
- Morning: Check new orders and SLA status
- Midday: Pack and print batch orders
- Afternoon: Arrange courier pickups
- Evening: Monitor shipping/delivery status
- Review metrics weekly

### Efficiency Tools

**Filter Combinations:**
- Date + SLA filters for urgent orders
- Product Name + Date for inventory planning
- Payment Channel + Order Type for accounting
- Print Status for document completion tracking

**Bulk Actions:**
- Process similar orders together
- Group by delivery area for courier efficiency
- Batch print to save time
- Export regularly for backup

**Search Optimization:**
- Use Order Number for customer inquiries
- Tracking Number for delivery follow-ups
- Customer Name for repeat buyer recognition
- Product Name for inventory correlation

---

## Common Issues and Solutions

### Issue: Orders Not Showing in To Ship Tab

**Possible Causes:**
- Orders still in Unpaid status
- Recently paid (takes 5-15 minutes to update)
- Filter settings hiding orders
- Orders already moved to next status

**Solutions:**
1. Check "Unpaid" tab for payment confirmation
2. Wait 15 minutes after payment notification
3. Click "More" and check all filters - click "Reset" if needed
4. Review "To Arrange Shipment" or "To Handover" tabs
5. Use Order Number search to locate specific order
6. Check "All" tab to see order current status

### Issue: Cannot Print Documents (Pack & Print Not Working)

**Possible Causes:**
- No orders selected
- Browser pop-up blocker active
- Printer not configured
- System temporary issue
- Document generation in progress

**Solutions:**
1. Ensure orders are selected (checkbox checked)
2. Check browser pop-up settings - allow for sellercenter.daraz.com.bd
3. Verify printer connection and settings
4. Try printing one order at a time if bulk fails
5. Refresh page and retry
6. Clear browser cache
7. Try different browser (Chrome recommended)
8. Contact Seller Support if persists

### Issue: SLA Breached - What Happens?

**Consequences:**
- Negative impact on seller performance rating
- Reduced visibility in search results
- Potential seller tier downgrade
- Customer may cancel without penalty
- Repeated breaches trigger review/warnings

**Prevention:**
1. Set alerts for approaching SLA deadlines
2. Pack orders within 12 hours of payment
3. Maintain adequate inventory levels
4. Pre-arrange regular courier pickups
5. Have backup courier options
6. Process orders even on weekends/holidays

**If Already Breached:**
1. Fulfill order as quickly as possible
2. Proactive customer communication
3. Apologize for delay
4. Offer discount on future order (optional)
5. Monitor closely to prevent recurring breaches
6. Analyze cause and implement fixes

### Issue: Customer Address Incorrect/Incomplete

**Immediate Actions:**
1. Contact customer via Daraz messaging BEFORE shipping
2. Request complete/corrected address
3. Verify new address format
4. Update in system if allowed
5. Document conversation

**If Already Shipped:**
1. Contact courier immediately
2. Request address update if possible
3. Inform customer of situation
4. If delivery fails, follow failed delivery workflow
5. Be prepared to reship or refund

**Prevention:**
- Review addresses before packing
- Flag unusual/incomplete addresses
- Contact customer proactively for clarifications
- Keep common address format examples

### Issue: Return Request Seems Unfair/Fraudulent

**Evaluation Steps:**
1. Review original order details
2. Check product condition claims
3. Compare timeline (returns must be within policy period)
4. Look for evidence of use vs. defect
5. Check customer return history

**Options:**
1. **Accept if Uncertain:** Customer satisfaction priority
2. **Reject with Clear Reason:** Document evidence
3. **Request Additional Information:** Photos, descriptions
4. **Escalate to Platform:** For fraudulent patterns
5. **Offer Partial Refund:** Compromise solution

**Protection:**
- Document product condition before shipping (photos)
- Include condition notes in packing
- Use tamper-evident packaging
- Clear return policy in listing
- Respond professionally even if rejecting

### Issue: High Failed Delivery Rate

**Analysis:**
1. Review failure reasons - identify patterns
2. Check specific courier performance
3. Analyze delivery areas with issues
4. Review customer communication practices

**Common Fixes:**

**Courier Issues:**
- Switch to more reliable courier for problem areas
- Report repeated courier failures to Daraz logistics
- Provide better delivery instructions to courier

**Customer Communication:**
- Send delivery notification with tracking
- Request confirmation of delivery date/time
- Provide courier contact information
- Ask customer to specify preferred delivery window

**Address Problems:**
- Verify address before shipping
- Add delivery landmarks in notes
- Coordinate with customer for complex locations
- Use alternative delivery points if needed

**System Solutions:**
- Enable SMS alerts for delivery
- Request customer phone verification
- Schedule deliveries for specific timeframes
- Use courier with better tracking systems

### Issue: Return Rate Too High

**Identify Root Causes:**
1. Export return data
2. Analyze by:
   - Product category
   - Return reasons
   - Time period
   - Specific items

**Common Causes & Fixes:**

**Wrong Expectations:**
- Improve product photos (multiple angles, details)
- Detailed specifications in description
- Clear size/dimension information
- Mention any limitations upfront
- Add video if complex product

**Quality Issues:**
- Review supplier quality
- Inspect items before shipping
- Better packaging to prevent damage
- Address manufacturing defects with supplier
- Consider alternative suppliers

**Size/Fit Issues (Apparel):**
- Comprehensive size charts
- Fit guides (slim, regular, loose)
- Model measurements in photos
- Customer measurement instructions
- Size comparison to popular brands

**Product Not as Described:**
- Accurate color representation
- Real product photos (not stock images)
- Honest condition description
- Mention any variations
- List what's included in box

### Issue: Cannot Find Specific Order

**Search Strategy:**
1. Try "All" tab first (searches across all statuses)
2. Use Order Number if customer provides it
3. Search by Customer Name
4. Search by Product Name if customer describes it
5. Use date range if approximate order date known
6. Search by Tracking Number if in shipping phase

**If Still Not Found:**
1. Check if order is in Return Orders page instead
2. Verify correct marketplace (Bangladesh vs others)
3. Check if order assigned to different seller account (multi-account sellers)
4. Contact Seller Support with order details
5. Request customer to provide order number from their account

---

## Important Notes

### Fulfillment SLA (Service Level Agreement)

**Standard SLA:**
- **To Pack:** 24 hours from payment confirmation
- **To Arrange Shipment:** 12 hours from packing completion
- **To Handover:** Must meet courier pickup schedule

**SLA Countdown:**
- Timer visible on order details
- Updates in real-time
- Includes weekends and holidays
- Extensions possible for force majeure (rare)

**Performance Tracking:**
- SLA adherence tracked monthly
- Affects seller tier
- Impacts search ranking
- Part of customer satisfaction score

### Payment Settlement

**Timeline:**
- Payment released after successful delivery
- Typical settlement: 7-14 days post-delivery
- COD orders: Settlement after courier remits payment
- Prepaid orders: Settlement after delivery confirmation

**Settlement Status:**
- Viewable in Finance section
- Pending settlements listed
- Completed settlements archived
- Settlement reports downloadable

### Return Policy Compliance

**Platform Policy:**
- 7-14 day return window (varies by category)
- Seller must honor valid returns
- Refund processed within 7 days of QC approval
- Non-return items: Personalized, intimate apparel, perishables

**Seller Responsibility:**
- Cannot reject valid returns
- Must process refunds promptly
- Responsible for return shipping if defective
- Customer pays return if change of mind (policy dependent)

### Document Requirements

**Mandatory Documents:**
- **AWB (Air Waybill):** Required for courier acceptance
- **Invoice:** Legal requirement for orders
- **PickList:** Optional but recommended for accuracy

**Document Retention:**
- Keep digital copies of all orders
- Export monthly order data
- Archive for at least 1 year
- Required for dispute resolution

### Multi-Channel Considerations

**Other Marketplaces:**
- Daraz operates in Pakistan, Bangladesh, Nepal, Myanmar, Sri Lanka
- Each marketplace separate order management
- Switch marketplace via footer links
- Orders don't transfer between countries

**Account Management:**
- Single sign-on across marketplaces
- Separate inventory per marketplace
- Different seller ratings per country
- Currency in local denomination

---

## Related Features

- [**Add Product**](../01-add-product/README.md) - List products for sale
- [**Manage Products**](../03-manage-products/README.md) - Update inventory and pricing
- [**Media Center**](../02-media-center/README.md) - Manage product images

---

**Last Updated:** January 18, 2026

**Feature Location:** Orders and Reviews → Orders

**Primary URL:** https://sellercenter.daraz.com.bd/apps/order/list

**Return Orders URL:** https://sellercenter.daraz.com.bd/apps/order/reverse
