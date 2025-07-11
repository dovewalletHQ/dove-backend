# Dove Beneficiary Management API Guide - Frontend Developers

## Overview for Frontend Developers

This guide provides the essential API information needed to implement the Dove beneficiary management system in your frontend application. The beneficiary system allows users to save frequently used recipients for easy money transfers, with automatic usage tracking and analytics.

---

## Beneficiary System Overview

### What are Beneficiaries?
Saved recipients that users can quickly select for transfers without re-entering account details each time.

### Key Features:
- **Save Recipients:** Store account details for frequent transfers
- **Usage Tracking:** Automatic statistics on transfer frequency and amounts  
- **Search & Filter:** Find beneficiaries by name, bank, or account number
- **Status Management:** Active/inactive beneficiary control
- **Analytics:** Usage statistics and insights

### API Endpoints Summary:
```
POST /api/v1/beneficiaries/           - Add new beneficiary
GET  /api/v1/beneficiaries/           - Get user's beneficiaries (paginated)
GET  /api/v1/beneficiaries/{id}       - Get single beneficiary details
PUT  /api/v1/beneficiaries/{id}       - Update beneficiary details
DELETE /api/v1/beneficiaries/{id}     - Delete beneficiary
GET  /api/v1/beneficiaries/stats      - Get beneficiary analytics
```

---

## API Endpoints Documentation

### 1. Add Beneficiary
Create a new beneficiary for the authenticated user.

**Endpoint:** `POST /api/v1/beneficiaries/`

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request:**
```json
{
  "name": "John Doe",                    // Required: Beneficiary full name (2-100 characters)
  "account_number": "1234567890",        // Required: 10-digit account number
  "bank_name": "Access Bank",            // Required: Full bank name (2-100 characters)
  "bank_code": "044",                    // Required: 3-digit bank code
  "nickname": "Johnny",                  // Optional: Nickname for easy identification (max 50 characters)
  "notes": "My brother",                 // Optional: Notes about beneficiary (max 200 characters)
  "beneficiary_type": "EXTERNAL"         // Optional: "INTERNAL" or "EXTERNAL" (default: "EXTERNAL")
}
```

**Success Response (200):**
```json
{
  "success": true,
  "responseCode": "200",
  "message": "Beneficiary added successfully",
  "data": {
    "id": "beneficiary_id_123",
    "name": "John Doe",
    "account_number": "1234567890",
    "bank_name": "Access Bank",
    "bank_code": "044",
    "nickname": "Johnny",
    "notes": "My brother",
    "beneficiary_type": "EXTERNAL",
    "status": "ACTIVE",
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:30:00Z",
    "last_used_at": null,
    "total_transfers": 0,
    "total_amount_sent": 0.0
  }
}
```

**Error Responses:**
- **400:** `{"success": false, "message": "Beneficiary with this account number and bank already exists", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Invalid account number format", "responseCode": "400"}`
- **401:** `{"detail": "Not authenticated"}`
- **500:** `{"success": false, "message": "Failed to add beneficiary", "responseCode": "500"}`

**Notes:**
- Name is automatically formatted to title case
- Account number must be exactly 10 digits
- Bank code must be exactly 3 digits
- Nickname and notes are optional (max 50 and 200 characters)
- Beneficiary type defaults to "EXTERNAL" if not specified
- Duplicate prevention: Same account number + bank code per user not allowed

---

### 2. Get User Beneficiaries
**Endpoint:** `GET /api/v1/beneficiaries/`

**Purpose:** Retrieve paginated list of user's beneficiaries with optional filtering

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Query Parameters:**
```
page=1              // Optional: Page number (default: 1)
page_size=20        // Optional: Items per page (default: 20)
status=ACTIVE       // Optional: "ACTIVE" or "INACTIVE"
search=john         // Optional: Search in name, nickname, bank name, account number
```

**Success Response (200):**
```json
{
  "success": true,
  "responseCode": "200",
  "message": "Beneficiaries retrieved successfully",
  "data": {
    "beneficiaries": [
      {
        "id": "beneficiary_id_123",
        "name": "John Doe",
        "account_number": "1234567890",
        "bank_name": "Access Bank",
        "bank_code": "044",
        "nickname": "Johnny",
        "notes": "My brother",
        "beneficiary_type": "EXTERNAL",
        "status": "ACTIVE",
        "created_at": "2024-01-15T10:30:00Z",
        "updated_at": "2024-01-15T10:30:00Z",
        "last_used_at": "2024-01-20T14:15:00Z",
        "total_transfers": 5,
        "total_amount_sent": 250000.0
      }
    ],
    "total": 25,
    "page": 1,
    "page_size": 20,
    "total_pages": 2
  }
}
```

**Error Responses:**
- **401:** `{"detail": "Not authenticated"}`
- **500:** `{"success": false, "message": "Failed to retrieve beneficiaries", "responseCode": "500"}`

**Notes:**
- Results are sorted by last usage date, then creation date
- Search works across name, nickname, bank name, and account number
- Pagination supports up to 100 items per page
- Status filter helps separate active from inactive beneficiaries

---

### 3. Get Single Beneficiary
**Endpoint:** `GET /api/v1/beneficiaries/{beneficiary_id}`

**Purpose:** Retrieve details of a specific beneficiary

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Success Response (200):**
```json
{
  "success": true,
  "responseCode": "200",
  "message": "Beneficiary retrieved successfully",
  "data": {
    "id": "beneficiary_id_123",
    "name": "John Doe",
    "account_number": "1234567890",
    "bank_name": "Access Bank",
    "bank_code": "044",
    "nickname": "Johnny",
    "notes": "My brother",
    "beneficiary_type": "EXTERNAL",
    "status": "ACTIVE",
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:30:00Z",
    "last_used_at": "2024-01-20T14:15:00Z",
    "total_transfers": 5,
    "total_amount_sent": 250000.0
  }
}
```

**Error Responses:**
- **404:** `{"success": false, "message": "Beneficiary not found", "responseCode": "404"}`
- **401:** `{"detail": "Not authenticated"}`
- **500:** `{"success": false, "message": "Failed to retrieve beneficiary", "responseCode": "500"}`

**Notes:**
- Only returns beneficiaries belonging to the authenticated user
- Includes complete usage statistics and metadata
- Use for displaying detailed beneficiary information

---

### 4. Update Beneficiary
**Endpoint:** `PUT /api/v1/beneficiaries/{beneficiary_id}`

**Purpose:** Update beneficiary details (name, nickname, notes, status only)

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request:**
```json
{
  "name": "John Smith",            // Optional: Updated beneficiary name (2-100 characters)
  "nickname": "John",              // Optional: Updated nickname (max 50 characters)
  "notes": "Updated notes",        // Optional: Updated notes (max 200 characters)
  "status": "INACTIVE"             // Optional: "ACTIVE" or "INACTIVE"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "responseCode": "200",
  "message": "Beneficiary updated successfully",
  "data": {
    "id": "beneficiary_id_123",
    "name": "John Smith",
    "account_number": "1234567890",
    "bank_name": "Access Bank",
    "bank_code": "044",
    "nickname": "John",
    "notes": "Updated notes",
    "beneficiary_type": "EXTERNAL",
    "status": "INACTIVE",
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-22T09:15:00Z",
    "last_used_at": "2024-01-20T14:15:00Z",
    "total_transfers": 5,
    "total_amount_sent": 250000.0
  }
}
```

**Error Responses:**
- **404:** `{"success": false, "message": "Beneficiary not found", "responseCode": "404"}`
- **401:** `{"detail": "Not authenticated"}`
- **500:** `{"success": false, "message": "Failed to update beneficiary", "responseCode": "500"}`

**Notes:**
- Account number and bank details cannot be modified
- Only provided fields will be updated
- Name will be auto-formatted to title case
- Updated timestamp is automatically set

---

### 5. Delete Beneficiary
**Endpoint:** `DELETE /api/v1/beneficiaries/{beneficiary_id}`

**Purpose:** Permanently delete a beneficiary

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Success Response (200):**
```json
{
  "success": true,
  "responseCode": "200",
  "message": "Beneficiary deleted successfully"
}
```

**Error Responses:**
- **404:** `{"success": false, "message": "Beneficiary not found", "responseCode": "404"}`
- **401:** `{"detail": "Not authenticated"}`
- **500:** `{"success": false, "message": "Failed to delete beneficiary", "responseCode": "500"}`

**Notes:**
- Deletion is permanent and cannot be undone
- Only the beneficiary owner can delete their beneficiaries
- All usage statistics are lost upon deletion

---

### 6. Get Beneficiary Statistics
**Endpoint:** `GET /api/v1/beneficiaries/stats`

**Purpose:** Retrieve user's beneficiary analytics and statistics

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Success Response (200):**
```json
{
  "success": true,
  "responseCode": "200",
  "message": "Beneficiary statistics retrieved successfully",
  "data": {
    "total_beneficiaries": 15,
    "active_beneficiaries": 12,
    "inactive_beneficiaries": 3,
    "most_used_beneficiary": {
      "id": "beneficiary_id_123",
      "name": "John Doe",
      "account_number": "1234567890",
      "bank_name": "Access Bank",
      "total_transfers": 25,
      "total_amount_sent": 500000.0
    },
    "recent_beneficiaries": [
      // Array of 5 most recently used beneficiaries with full details
    ]
  }
}
```

**Error Responses:**
- **401:** `{"detail": "Not authenticated"}`
- **500:** `{"success": false, "message": "Failed to retrieve beneficiary statistics", "responseCode": "500"}`

**Notes:**
- Most used beneficiary is determined by total transfer count
- Recent beneficiaries are sorted by last usage date
- Statistics include both active and inactive beneficiaries
- Useful for analytics dashboards and insights


---

## Authentication & Security

### **Token Management:**
- **Token Type:** JWT Bearer token
- **Usage:** Include in `Authorization` header for all endpoints
- **Format:** `Authorization: Bearer YOUR_ACCESS_TOKEN`

### **Security Notes:**
- All endpoints require authentication
- Tokens should be stored securely
- Always use HTTPS for API communications
- Account details are automatically validated and formatted

---

## Error Handling Guide

### **Common Error Scenarios:**

**Authentication Errors:**
- **401 Unauthorized:** `{"detail": "Not authenticated"}`

**Validation Errors:**
- **Invalid Account Number:** Account number must be exactly 10 digits
- **Invalid Bank Code:** Bank code must be exactly 3 digits
- **Invalid Name:** Name must be 2-100 characters
- **Duplicate Beneficiary:** Same account + bank combination already exists

**Server Errors:**
- **500 Internal Error:** Backend processing failed

### **Error Response Format:**
All error responses follow this structure:
```json
{
  "success": false,
  "message": "Human readable error message",
  "responseCode": "400"
}
```

---

## Usage & Analytics

### **Automatic Usage Tracking:**
When transfers are made through the wallet system to saved beneficiaries:
- `total_transfers` is automatically incremented
- `total_amount_sent` is updated with transfer amount
- `last_used_at` is set to current timestamp
- No additional frontend implementation required

### **Analytics Features:**
- Total, active, and inactive beneficiary counts
- Most frequently used beneficiary identification
- Recent beneficiaries based on last usage
- Transfer statistics per beneficiary

---

## Validation Requirements

### **Required Field Formats:**
- **Name:** 2-100 characters (auto-formatted to title case)
- **Account Number:** Exactly 10 digits (numeric only)
- **Bank Name:** 2-100 characters
- **Bank Code:** Exactly 3 digits (numeric only)
- **Nickname:** Max 50 characters (optional)
- **Notes:** Max 200 characters (optional)
- **Beneficiary Type:** "INTERNAL" or "EXTERNAL" (default: "EXTERNAL")
- **Status:** "ACTIVE" or "INACTIVE" (default: "ACTIVE")

### **Frontend Validation Tips:**
- Validate account numbers as 10-digit strings
- Validate bank codes as 3-digit strings
- Check character limits before submission
- Handle duplicate prevention gracefully

---

## Integration with Transfer Flow

### **Beneficiary Selection:**
When integrating with transfer/payment flows:
1. Load user's active beneficiaries
2. Allow selection from saved beneficiaries
3. Pre-fill transfer form with beneficiary details
4. Include beneficiary ID for automatic usage tracking

### **Transfer Integration Data:**
```json
{
  "recipientName": "beneficiary.name",
  "accountNumber": "beneficiary.account_number", 
  "bankName": "beneficiary.bank_name",
  "bankCode": "beneficiary.bank_code",
  "beneficiaryId": "beneficiary.id"
}
```

---

## Performance & Caching

### **Recommended Practices:**
- Cache beneficiary lists for 5-10 minutes
- Implement search debouncing (300ms delay)
- Use pagination for large beneficiary lists
- Show loading states during API calls
- Refresh cache after add/update/delete operations

### **Expected Response Times:**
- **Add Beneficiary:** 1-3 seconds
- **Get Beneficiaries:** 1-2 seconds
- **Update/Delete:** 1-2 seconds
- **Get Statistics:** 2-3 seconds

---

## Mobile App Considerations

### **User Experience:**
- Quick beneficiary selection during transfers
- Search functionality for large beneficiary lists
- Display usage statistics (transfer count, amounts)
- Show most recent and most used beneficiaries
- Allow easy editing of nicknames and notes

### **Data Management:**
- Cache frequently used beneficiaries locally
- Sync with backend periodically
- Handle offline scenarios gracefully
- Provide clear loading and error states

---

## Development vs Production

### **Environment Considerations:**
- Test with various account number formats
- Validate error handling scenarios
- Test pagination with large datasets
- Verify authentication token handling
- Test network timeout scenarios

---

This guide provides all the API information needed to implement beneficiary management with automatic usage tracking and analytics. The backend handles all validation, security, and business logic - focus on creating a smooth user experience for saving and selecting frequent transfer recipients!