# Refund Handling System Documentation

## Overview
The Golden Palm Resort refund handling system provides comprehensive functionality for managing refund requests, approvals, and processing. The system includes a complete workflow from refund request creation to final processing.

## System Components

### 1. Data Model (`Refund.java`)
- **Entity**: `com.sliit.goldenpalmresort.model.Refund`
- **Table**: `refunds`
- **Key Features**:
  - Tracks refund amount, reason, and method
  - Status management (PENDING, APPROVED, PROCESSING, COMPLETED, REJECTED, CANCELLED)
  - Audit trail with created/updated timestamps
  - Links to original payment
  - Approval and processing tracking

### 2. Repository Layer (`RefundRepository.java`)
- **Interface**: `com.sliit.goldenpalmresort.repository.RefundRepository`
- **Features**:
  - Standard CRUD operations
  - Custom queries for status-based filtering
  - Statistical aggregations
  - Date range filtering
  - Performance-optimized indexes

### 3. Service Layer (`RefundService.java`)
- **Class**: `com.sliit.goldenpalmresort.service.RefundService`
- **Features**:
  - Business logic for refund processing
  - Validation of refund eligibility
  - Status transition management
  - Payment status updates
  - Comprehensive error handling

### 4. Controller Layer
#### RefundController (`RefundController.java`)
- **Endpoint**: `/api/refunds`
- **Features**:
  - RESTful API for refund management
  - Complete CRUD operations
  - Status management endpoints
  - Statistics and reporting

#### PaymentOfficerController (Enhanced)
- **Endpoint**: `/api/payment-officer/refunds`
- **Features**:
  - Payment officer specific refund operations
  - Integration with existing payment system
  - Enhanced reporting capabilities

### 5. DTOs
#### RefundRequest (`RefundRequest.java`)
- Input validation
- Required field validation
- Size constraints
- Data type validation

#### RefundResponse (`RefundResponse.java`)
- Comprehensive refund information
- Payment details
- Customer information
- Booking references

### 6. Database Migration
- **File**: `V3__create_refunds_table.sql`
- **Features**:
  - Complete table structure
  - Foreign key constraints
  - Performance indexes
  - Audit columns

### 7. Web Interface
- **File**: `refund-management.html`
- **Features**:
  - Modern, responsive design
  - Tabbed interface
  - Real-time data loading
  - Interactive refund management
  - Statistics dashboard

## API Endpoints

### Refund Management Endpoints

#### Create Refund Request
```
POST /api/refunds
Content-Type: application/json

{
  "paymentId": 123,
  "refundAmount": 150.00,
  "refundReason": "Customer cancellation",
  "refundMethod": "ORIGINAL_PAYMENT_METHOD",
  "processedBy": "John Doe",
  "notes": "Additional notes"
}
```

#### Get All Refunds
```
GET /api/refunds
```

#### Get Refund by ID
```
GET /api/refunds/{id}
```

#### Get Refunds by Status
```
GET /api/refunds/status/{status}
```

#### Approve Refund
```
PUT /api/refunds/{id}/approve
Content-Type: application/json

{
  "approvedBy": "Manager Name"
}
```

#### Reject Refund
```
PUT /api/refunds/{id}/reject
Content-Type: application/json

{
  "processedBy": "Manager Name",
  "reason": "Invalid request"
}
```

#### Process Refund
```
PUT /api/refunds/{id}/process
Content-Type: application/json

{
  "processedBy": "Payment Officer"
}
```

#### Get Refund Statistics
```
GET /api/refunds/statistics
```

### Payment Officer Endpoints

#### Create Refund Request (Payment Officer)
```
POST /api/payment-officer/refunds
```

#### Get All Refunds (Payment Officer)
```
GET /api/payment-officer/refunds
```

#### Get Refunds Requiring Approval
```
GET /api/payment-officer/refunds/pending-approval
```

## Refund Workflow

### 1. Refund Request Creation
- Customer or staff creates refund request
- System validates payment eligibility
- Validates refund amount against payment amount
- Creates refund record with PENDING status

### 2. Approval Process
- Manager reviews refund request
- Can approve or reject the request
- System updates status to APPROVED or REJECTED
- Audit trail maintained

### 3. Processing
- Payment officer processes approved refunds
- Status updated to PROCESSING
- Actual refund processing occurs
- Status updated to COMPLETED

### 4. Payment Status Updates
- System automatically updates original payment status
- Full refund: Payment status → REFUNDED
- Partial refund: Payment status → PARTIALLY_REFUNDED

## Database Schema

### Refunds Table
```sql
CREATE TABLE refunds (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    payment_id BIGINT NOT NULL,
    refund_amount DECIMAL(10,2) NOT NULL,
    refund_status VARCHAR(20) NOT NULL DEFAULT 'PENDING',
    refund_reason VARCHAR(500),
    refund_method VARCHAR(50),
    processed_by VARCHAR(100),
    approved_by VARCHAR(100),
    refund_date TIMESTAMP,
    processed_date TIMESTAMP,
    notes VARCHAR(1000),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (payment_id) REFERENCES payments(id) ON DELETE CASCADE
);
```

## Status Definitions

### Refund Statuses
- **PENDING**: Initial status when refund is requested
- **APPROVED**: Refund has been approved by manager
- **PROCESSING**: Refund is being processed by payment officer
- **COMPLETED**: Refund has been successfully processed
- **REJECTED**: Refund request has been rejected
- **CANCELLED**: Refund request has been cancelled

## Security Considerations

### Validation
- Refund amount validation against payment amount
- Payment eligibility checks
- Status transition validation
- User authorization for operations

### Audit Trail
- Complete audit trail for all refund operations
- User tracking for approvals and processing
- Timestamp tracking for all operations

## Error Handling

### Common Error Scenarios
1. **Invalid Payment ID**: Payment not found
2. **Invalid Refund Amount**: Amount exceeds payment amount
3. **Invalid Status Transition**: Attempting invalid status changes
4. **Unauthorized Operations**: Insufficient permissions

### Error Responses
```json
{
  "error": "Error message description"
}
```

## Performance Considerations

### Database Indexes
- Primary key on `id`
- Index on `payment_id` for foreign key lookups
- Index on `refund_status` for status filtering
- Index on `created_at` for date range queries
- Composite indexes for common query patterns

### Query Optimization
- Efficient status-based filtering
- Optimized statistical queries
- Proper use of database indexes
- Minimal N+1 query problems

## Testing Recommendations

### Unit Tests
- Service layer business logic
- Repository query methods
- Validation logic
- Status transition logic

### Integration Tests
- API endpoint testing
- Database integration
- End-to-end workflow testing
- Error scenario testing

### Performance Tests
- Large dataset handling
- Concurrent request processing
- Database query performance
- Memory usage optimization

## Deployment Notes

### Database Migration
1. Run migration script: `V3__create_refunds_table.sql`
2. Verify table creation and indexes
3. Test foreign key constraints

### Application Configuration
1. Ensure proper database connection
2. Configure JPA entity scanning
3. Set up proper logging levels
4. Configure CORS if needed

## Usage Examples

### Creating a Refund Request
```javascript
const refundData = {
    paymentId: 123,
    refundAmount: 150.00,
    refundReason: "Customer cancellation",
    refundMethod: "ORIGINAL_PAYMENT_METHOD",
    processedBy: "John Doe",
    notes: "Customer requested cancellation"
};

fetch('/api/refunds', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(refundData)
});
```

### Approving a Refund
```javascript
fetch('/api/refunds/456/approve', {
    method: 'PUT',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({
        approvedBy: "Manager Name"
    })
});
```

## Future Enhancements

### Potential Improvements
1. **Email Notifications**: Automatic email notifications for status changes
2. **Bulk Operations**: Bulk refund processing capabilities
3. **Advanced Reporting**: Detailed refund analytics and reporting
4. **Integration**: Integration with external payment processors
5. **Automation**: Automated refund processing for certain scenarios
6. **Audit Reports**: Comprehensive audit and compliance reporting

### Scalability Considerations
1. **Caching**: Implement caching for frequently accessed data
2. **Pagination**: Add pagination for large datasets
3. **Background Processing**: Implement background job processing
4. **Microservices**: Consider breaking into microservices for large scale

## Support and Maintenance

### Monitoring
- Monitor refund processing times
- Track approval rates and rejection reasons
- Monitor system performance and errors
- Regular database maintenance and optimization

### Maintenance Tasks
- Regular database cleanup of old records
- Performance monitoring and optimization
- Security updates and patches
- Backup and recovery procedures

This refund handling system provides a comprehensive solution for managing refunds in the Golden Palm Resort application, with proper validation, audit trails, and user-friendly interfaces.
