# Business Logic Analysis

## Overview

This document provides a comprehensive analysis of the business logic components in the VB6 application, including business rules, workflows, calculations, and validation logic that must be preserved during migration to .NET Core.

## Business Domain Analysis

### Core Business Entities

#### Customer Entity
```
Customer Properties:
- CustomerID: Unique identifier (Auto-generated)
- CompanyName: Business name (Required, 2-100 chars)
- ContactName: Primary contact (Required, 2-50 chars)
- ContactTitle: Job title (Optional, 2-50 chars)
- Address: Street address (Required, 10-200 chars)
- City: City name (Required, 2-50 chars)
- Region: State/Province (Optional, 2-50 chars)
- PostalCode: ZIP/Postal code (Required, format validation)
- Country: Country name (Required, 2-50 chars)
- Phone: Phone number (Required, format validation)
- Fax: Fax number (Optional, format validation)
- Email: Email address (Required, format validation)
- CustomerType: Individual/Corporate (Required)
- CreditLimit: Credit amount (Optional, > 0)
- IsActive: Active status (Required, boolean)
- CreatedDate: Creation timestamp (Auto-generated)
- ModifiedDate: Last modification (Auto-generated)
```

#### Order Entity
```
Order Properties:
- OrderID: Unique identifier (Auto-generated)
- CustomerID: Foreign key to Customer (Required)
- EmployeeID: Foreign key to Employee (Required)
- OrderDate: Order placement date (Required)
- RequiredDate: Requested delivery date (Optional)
- ShippedDate: Actual shipping date (Optional)
- ShipVia: Shipping method ID (Required)
- Freight: Shipping cost (Calculated, >= 0)
- ShipName: Shipping recipient name (Required)
- ShipAddress: Shipping street address (Required)
- ShipCity: Shipping city (Required)
- ShipRegion: Shipping region (Optional)
- ShipPostalCode: Shipping postal code (Required)
- ShipCountry: Shipping country (Required)
- OrderStatus: Order status enum (Required)
- TotalAmount: Order total (Calculated)
- TaxAmount: Tax amount (Calculated)
- DiscountAmount: Discount amount (Calculated)
```

#### Product Entity
```
Product Properties:
- ProductID: Unique identifier (Auto-generated)
- ProductName: Product name (Required, 2-100 chars)
- SupplierID: Foreign key to Supplier (Required)
- CategoryID: Foreign key to Category (Required)
- QuantityPerUnit: Packaging description (Optional)
- UnitPrice: Sale price (Required, > 0)
- UnitsInStock: Current stock (Required, >= 0)
- UnitsOnOrder: Ordered quantity (Required, >= 0)
- ReorderLevel: Reorder threshold (Required, >= 0)
- Discontinued: Discontinued status (Required, boolean)
- CreatedDate: Creation timestamp (Auto-generated)
- ModifiedDate: Last modification (Auto-generated)
```

### Business Rules Catalog

#### Customer Management Rules

##### Customer Creation Rules
```
Validation Rules:
1. CompanyName: Required, unique, 2-100 characters, alphanumeric + spaces + punctuation
2. ContactName: Required, 2-50 characters, alphabetic + spaces
3. Email: Required, valid email format, unique across all customers
4. Phone: Required, valid phone format (10-15 digits, various formats accepted)
5. PostalCode: Required, valid format for country (US ZIP: 12345 or 12345-6789)
6. CreditLimit: Optional, if provided must be >= 0, max based on customer type

Business Rules:
1. Email uniqueness check across all active customers
2. Automatic customer type assignment based on annual revenue (if available)
3. Credit limit validation based on customer type and payment history
4. Address validation for tax jurisdiction purposes
5. Duplicate detection based on company name and contact information
```

##### Customer Update Rules
```
Validation Rules:
1. All creation rules apply except uniqueness constraints
2. Email changes require verification (future enhancement)
3. Credit limit changes require manager approval for increases > 20%
4. Address changes may affect tax calculations

Business Rules:
1. Maintain audit trail of all changes
2. Recalculate order eligibility based on credit limit changes
3. Update shipping addresses on open orders if requested
4. Notify sales team of significant changes (credit limit, contact info)
```

#### Order Processing Rules

##### Order Creation Rules
```
Validation Rules:
1. CustomerID: Must exist and be active
2. At least one order item required
3. All items must be in stock (quantity <= UnitsInStock)
4. Unit price must match current product price
5. Quantity must be > 0 and <= max order quantity per item
6. RequiredDate must be >= OrderDate + minimum lead time
7. Shipping address required and valid

Business Rules:
1. Calculate order totals (subtotal, tax, shipping, discount)
2. Apply customer-specific pricing (volume discounts, special pricing)
3. Check credit limit against order total + outstanding balance
4. Reserve inventory for order items
5. Calculate estimated shipping cost based on weight and distance
6. Apply promotional discounts if applicable
7. Generate order number with business rules (sequential, prefixed)
```

##### Order Status Workflow
```
Order States:
1. New: Order created, pending approval
2. Approved: Order approved, ready for processing
3. Processing: Order being prepared for shipment
4. Shipped: Order shipped, tracking number assigned
5. Delivered: Order delivered to customer
6. Cancelled: Order cancelled
7. Returned: Order returned by customer

State Transition Rules:
- New → Approved: Automatic if under credit limit, manual approval otherwise
- New → Cancelled: Customer or system cancellation
- Approved → Processing: Inventory allocation successful
- Processing → Shipped: Shipping arranged, tracking number obtained
- Shipped → Delivered: Delivery confirmation received
- Any State → Returned: Return request processed
- Any State → Cancelled: Cancellation request approved
```

#### Inventory Management Rules

##### Stock Level Rules
```
Validation Rules:
1. UnitsInStock: Must be >= 0
2. UnitsOnOrder: Must be >= 0
3. ReorderLevel: Must be >= 0 and <= max stock level

Business Rules:
1. Automatic reorder when UnitsInStock <= ReorderLevel
2. Stock allocation for orders reduces UnitsInStock
3. Stock receipt increases UnitsInStock, decreases UnitsOnOrder
4. Negative stock not allowed (backorders require approval)
5. Stock adjustments require audit trail and approval
6. Seasonal stock level adjustments
```

##### Product Pricing Rules
```
Validation Rules:
1. UnitPrice: Must be > 0
2. Price changes must be approved for decreases > 10%
3. Special pricing requires manager approval

Business Rules:
1. Volume-based pricing tiers
2. Customer-specific pricing agreements
3. Promotional pricing with time limits
4. Currency conversion for international customers
5. Price rounding rules (nearest cent)
6. Tax-inclusive vs tax-exclusive pricing
```

### Business Calculations

#### Order Total Calculation
```
Algorithm: CalculateOrderTotal(orderItems, customer, shippingInfo)
1. Calculate Subtotal
   subtotal = Σ(orderItem.quantity * orderItem.unitPrice)

2. Apply Volume Discounts
   if subtotal >= 1000 then discount = 0.05
   if subtotal >= 5000 then discount = 0.10
   if subtotal >= 10000 then discount = 0.15

3. Apply Customer-Specific Discounts
   customerDiscount = GetCustomerDiscount(customer.type, customer.loyaltyLevel)
   totalDiscount = discount + customerDiscount

4. Calculate Tax
   taxableAmount = subtotal * (1 - totalDiscount)
   taxRate = GetTaxRate(shippingInfo.state, shippingInfo.city)
   taxAmount = taxableAmount * taxRate

5. Calculate Shipping
   shippingWeight = Σ(orderItem.quantity * product.weight)
   shippingCost = CalculateShippingCost(shippingWeight, shippingInfo.distance, shippingMethod)

6. Calculate Total
   total = subtotal - (subtotal * totalDiscount) + taxAmount + shippingCost

7. Apply Final Adjustments
   if customer.hasSpecialPricing then
       total = ApplySpecialPricing(total, customer)
   end if

Return total
```

#### Credit Limit Validation
```
Algorithm: ValidateCreditLimit(customer, orderTotal)
1. Get Current Outstanding Balance
   outstanding = GetOutstandingBalance(customer.id)

2. Calculate Available Credit
   availableCredit = customer.creditLimit - outstanding

3. Apply Credit Rules
   if customer.type = "Premium" then
       availableCredit += 1000  // Premium buffer
   end if

   if customer.paymentTerms = "Net30" then
       availableCredit += orderTotal * 0.1  // Terms buffer
   end if

4. Validate Order
   if orderTotal > availableCredit then
       if orderTotal <= availableCredit * 1.1 then  // 10% grace
           RequireApproval = true
       else
           RejectOrder = true
       end if
   end if

Return validationResult
```

#### Inventory Allocation
```
Algorithm: AllocateInventory(orderItems)
1. Sort Items by Priority
   // Critical items first, then by value descending

2. For Each Order Item
   availableStock = GetAvailableStock(item.productId)

   if item.quantity > availableStock then
       if AllowBackorder(item.productId) then
           AllocatePartial(item, availableStock)
           BackorderRemaining(item, item.quantity - availableStock)
       else
           RejectItem(item, "Insufficient stock")
       end if
   else
       AllocateFull(item)
   end if

3. Update Stock Levels
   UpdateUnitsInStock(item.productId, -item.quantity)

4. Reserve Allocation
   CreateStockReservation(orderId, item.productId, item.quantity)

Return allocationResult
```

### Validation Logic

#### Input Validation Rules

##### Customer Validation
```vb
Function ValidateCustomer(customer As Customer) As ValidationResult
    Dim result As New ValidationResult

    ' Name validation
    If Len(Trim(customer.CompanyName)) < 2 Then
        result.AddError("Company name must be at least 2 characters")
    End If

    If Len(customer.CompanyName) > 100 Then
        result.AddError("Company name cannot exceed 100 characters")
    End If

    ' Email validation
    If Not IsValidEmail(customer.Email) Then
        result.AddError("Invalid email format")
    End If

    ' Phone validation
    If Not IsValidPhone(customer.Phone) Then
        result.AddError("Invalid phone number format")
    End If

    ' Postal code validation
    If Not IsValidPostalCode(customer.PostalCode, customer.Country) Then
        result.AddError("Invalid postal code for specified country")
    End If

    ' Business rule validation
    If CustomerExists(customer.Email) Then
        result.AddError("Customer with this email already exists")
    End If

    Return result
End Function
```

##### Order Validation
```vb
Function ValidateOrder(order As Order) As ValidationResult
    Dim result As New ValidationResult

    ' Customer validation
    If Not CustomerExists(order.CustomerID) Then
        result.AddError("Invalid customer")
    End If

    If Not IsActiveCustomer(order.CustomerID) Then
        result.AddError("Customer account is inactive")
    End If

    ' Date validation
    If order.RequiredDate < Date Then
        result.AddError("Required date cannot be in the past")
    End If

    If order.RequiredDate > DateAdd("d", 90, order.OrderDate) Then
        result.AddError("Required date cannot be more than 90 days from order date")
    End If

    ' Item validation
    If order.OrderItems.Count = 0 Then
        result.AddError("Order must contain at least one item")
    End If

    For Each item In orderItems
        If Not ProductExists(item.ProductID) Then
            result.AddError("Invalid product: " & item.ProductID)
        End If

        If item.Quantity <= 0 Then
            result.AddError("Invalid quantity for product: " & item.ProductID)
        End If

        If item.Quantity > GetAvailableStock(item.ProductID) Then
            result.AddError("Insufficient stock for product: " & item.ProductID)
        End If
    Next

    ' Credit validation
    Dim orderTotal As Double = CalculateOrderTotal(order)
    If Not ValidateCreditLimit(order.CustomerID, orderTotal) Then
        result.AddError("Order exceeds credit limit")
    End If

    Return result
End Function
```

### Workflow Analysis

#### Order Processing Workflow
```
1. Order Creation
   ├── Customer places order (manual/web service)
   ├── Validate order data
   ├── Check inventory availability
   ├── Validate credit limit
   ├── Calculate pricing and totals
   └── Create order record

2. Order Approval
   ├── Automatic approval (under credit limit)
   ├── Manual approval required (over limit)
   ├── Manager review and approval
   └── Order status update

3. Order Processing
   ├── Allocate inventory
   ├── Generate picking list
   ├── Prepare shipping documents
   ├── Arrange shipping
   └── Update order status

4. Order Shipping
   ├── Package items
   ├── Generate shipping label
   ├── Update tracking information
   ├── Send shipping notification
   └── Update order status

5. Order Delivery
   ├── Receive delivery confirmation
   ├── Update delivery date
   ├── Process payment (if COD)
   └── Close order

6. Order Completion
   ├── Archive order data
   ├── Update customer history
   ├── Generate sales reports
   └── Clean up temporary data
```

#### Customer Onboarding Workflow
```
1. Initial Contact
   ├── Lead qualification
   ├── Initial information gathering
   └── Lead assignment to sales rep

2. Information Collection
   ├── Detailed company information
   ├── Contact information validation
   ├── Credit application
   └── Reference checks

3. Credit Review
   ├── Credit score verification
   ├── Reference validation
   ├── Credit limit determination
   └── Approval process

4. Account Setup
   ├── Create customer record
   ├── Assign customer number
   ├── Set up pricing terms
   └── Configure shipping preferences

5. Activation
   ├── Send welcome package
   ├── Training session scheduling
   ├── Initial order assistance
   └── Account activation
```

### Integration Points

#### External System Integrations
```
1. Shipping Carrier Integration
   ├── Rate shopping
   ├── Label generation
   ├── Tracking updates
   └── Delivery confirmation

2. Payment Processing
   ├── Credit card processing
   ├── ACH payments
   ├── Payment gateway integration
   └── Fraud detection

3. Email System
   ├── Order confirmations
   ├── Shipping notifications
   ├── Payment receipts
   └── Marketing communications

4. Accounting System
   ├── Invoice generation
   ├── Payment posting
   ├── AR/AP management
   └── Financial reporting

5. Warehouse Management
   ├── Inventory synchronization
   ├── Order fulfillment
   ├── Shipping coordination
   └── Returns processing
```

### Error Handling and Recovery

#### Business Logic Error Scenarios
```
1. Inventory Allocation Failures
   ├── Insufficient stock
   ├── Concurrent allocation conflicts
   ├── Product discontinuation
   └── Allocation rollback procedures

2. Credit Limit Violations
   ├── Over limit orders
   ├── Payment failures
   ├── Credit limit adjustments
   └── Approval workflow triggers

3. Data Validation Errors
   ├── Invalid customer data
   ├── Incorrect product information
   ├── Address validation failures
   └── Data correction procedures

4. System Integration Failures
   ├── Shipping system downtime
   ├── Payment gateway issues
   ├── Email delivery failures
   └── Recovery and retry logic
```

### Performance Considerations

#### Business Logic Performance Requirements
```
1. Order Processing: < 5 seconds for standard orders
2. Customer Search: < 2 seconds for result display
3. Inventory Lookup: < 1 second for stock checks
4. Report Generation: < 30 seconds for standard reports
5. Batch Processing: < 10 minutes for 1000 orders

Optimization Strategies:
1. Database query optimization
2. Caching of reference data
3. Asynchronous processing for heavy calculations
4. Batch processing for bulk operations
5. Indexing strategy for search operations
```

### Testing Scenarios

#### Business Logic Test Cases
```
Unit Test Cases:
1. Order total calculation with various discount scenarios
2. Credit limit validation with different customer types
3. Inventory allocation with stock constraints
4. Customer validation with edge cases
5. Tax calculation for different jurisdictions

Integration Test Cases:
1. Complete order processing workflow
2. Customer onboarding process
3. Inventory management with order fulfillment
4. Payment processing integration
5. Shipping system coordination

Edge Cases:
1. Zero dollar orders
2. Maximum quantity orders
3. International address validation
4. Leap year date calculations
5. Currency conversion accuracy
```

## Migration Considerations

### Business Logic Preservation
```
Critical Requirements:
1. 100% accuracy in calculations
2. Preservation of all business rules
3. Maintenance of workflow sequences
4. Retention of validation logic
5. Preservation of error handling

Migration Strategy:
1. Extract and document all business rules
2. Create comprehensive unit tests
3. Implement business logic in domain layer
4. Validate results against VB6 system
5. Performance testing and optimization
```

### Modernization Opportunities
```
Business Logic Improvements:
1. Rule engine implementation for complex logic
2. Event-driven architecture for workflows
3. Microservice decomposition for scalability
4. API-first design for integration
5. Real-time processing capabilities

Architecture Enhancements:
1. Domain-Driven Design implementation
2. CQRS pattern for complex domains
3. Event sourcing for audit trails
4. Saga pattern for distributed transactions
5. Business process management integration
```

This analysis provides the foundation for ensuring that all critical business functionality is preserved and potentially enhanced during the migration to .NET Core.