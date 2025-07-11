# Dove Wallet Operations - Complete Developer Guide

## Table of Contents

### 1. [Overview & Introduction](#1-overview--introduction)
- [System Architecture](#system-architecture)
- [Authentication Requirements](#authentication-requirements)
- [API Base Information](#api-base-information)

### 2. [Naira Wallet Operations](#2-naira-wallet-operations)
- [Available Operations](#available-operations)
- [API Endpoints Summary](#api-endpoints-summary)
- [Wallet Information & Status](#wallet-information--status)
- [Transaction History](#transaction-history)
- [Bank Management](#bank-management)

### 3. [Transfer Operations](#3-transfer-operations)
- [Internal Transfers (Dove-to-Dove)](#internal-transfers-dove-to-dove)
  - [Account-Based Internal Transfers](#account-based-internal-transfers)
  - [Username-Based Internal Transfers](#username-based-internal-transfers)
  - [Username Lookup & Validation](#username-lookup--validation)
- [External Transfers (Other Banks)](#external-transfers-other-banks)
- [Transaction Status & Tracking](#transaction-status--tracking)

### 4. [Beneficiary Management](#4-beneficiary-management)
- [Beneficiary System Overview](#beneficiary-system-overview)
- [Regular Beneficiaries](#regular-beneficiaries)
- [Username-Based Internal Beneficiaries](#username-based-internal-beneficiaries)
- [Beneficiary Analytics & Statistics](#beneficiary-analytics--statistics)

### 5. [Integration Guidelines](#5-integration-guidelines)
- [Complete Transfer Workflows](#complete-transfer-workflows)
- [Fee Structure](#fee-structure)
- [Error Handling Guide](#error-handling-guide)
- [Security & Validation](#security--validation)

### 6. [Development Resources](#6-development-resources)
- [Mobile App Implementation Tips](#mobile-app-implementation-tips)
- [Testing & Development](#testing--development)
- [Performance & Caching](#performance--caching)
- [Monitoring & Logging](#monitoring--logging)

### 7. [Future Expansion](#7-future-expansion)
- [Multi-Currency Support (Coming Soon)](#multi-currency-support-coming-soon)
- [Support & Troubleshooting](#support--troubleshooting)

---

## 1. Overview & Introduction

### System Architecture

This comprehensive guide covers all wallet operations for the Dove mobile application, currently supporting Nigerian Naira (NGN) operations through 9PSB (Nine Payment Service Bank). The system supports both internal transfers between Dove users and external transfers to other Nigerian banks.

**Current Supported Currencies:**
- Nigerian Naira (NGN) - Full support

**Planned Currencies:**
- USD and other currencies - Coming soon

### Authentication Requirements

**Headers Required for ALL Endpoints:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Token Management:**
- All wallet operations require valid JWT token
- Token obtained from login/registration process
- Wallet operations use WAAS (Wallet-as-a-Service) scope

### API Base Information

**Base URL Structure:**
- Wallet Operations: `/api/v1/wallet/`
- Beneficiary Management: `/api/v1/beneficiaries/`

---

## 2. Naira Wallet Operations

### Available Operations

```
1. Wallet Info & Status → 2. Transaction History → 3. Bank List → 4. Account Verification → 5. Internal Transfers (Dove-to-Dove) → 6. External Transfers (Dove-to-Other Banks) → 7. Transaction Tracking → 8. Beneficiary Management
```

### API Endpoints Summary

```
GET  /api/v1/wallet/                    - Get wallet information
POST /api/v1/wallet/status              - Get wallet status
POST /api/v1/wallet/wallet_transactions - Get transaction history
POST /api/v1/wallet/get_banks           - Get list of Nigerian banks
POST /api/v1/wallet/other_bank_enquiry  - Verify recipient account
POST /api/v1/wallet/debit/transfer      - Internal wallet debit
POST /api/v1/wallet/credit/transfer     - Internal wallet credit
POST /api/v1/wallet/internal_transfer   - Username-based internal transfer
POST /api/v1/wallet/lookup_username     - Validate username for transfers
POST /api/v1/wallet/wallet_other_banks  - Transfer to other banks
POST /api/v1/wallet/wallet_requery      - Query transaction status
```

### Wallet Information & Status

#### Get Wallet Information
**Endpoint:** `GET /api/v1/wallet/`

**Purpose:** Retrieve current user's wallet details and balance

**Request:**
```json
{}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Wallet retrieved successfully",
  "responseCode": "200",
  "data": {
    "accountNumber": "2234567890",
    "accountName": "John William Doe",
    "balance": "5000.00",
    "currency": "NGN",
    "status": "ACTIVE",
    "bankName": "9 PAYMENT SERVICE BANK",
    "customerId": "CUST123456"
  }
}
```

**Error Responses:**
- **401:** `{"success": false, "message": "Unauthorized", "responseCode": "401"}`
- **201:** `{"success": false, "message": "User has no nuban account yet", "responseCode": "201"}`

**Notes:**
- Uses user's stored `nuban_payment_id` from profile
- Returns current balance and account status
- Required before performing any transfers

#### Get Wallet Status
**Endpoint:** `POST /api/v1/wallet/status`

**Purpose:** Check detailed wallet status and limits

**Request:**
```json
{}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Wallet status retrieved",
  "responseCode": "200",
  "data": {
    "accountStatus": "ACTIVE",
    "tier": "TIER_1",
    "dailyLimit": "50000.00",
    "monthlyLimit": "200000.00",
    "availableBalance": "5000.00"
  }
}
```

**Notes:**
- Automatically uses current user's account number
- Shows transaction limits based on KYC level
- Check before large transfers

### Transaction History

**Endpoint:** `POST /api/v1/wallet/wallet_transactions`

**Purpose:** Retrieve wallet transaction history with date filtering

**Request:**
```json
{
  "fromDate": "2024-01-01",
  "toDate": "2024-01-31",
  "numberOfItems": "50"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Transactions retrieved successfully",
  "responseCode": "200",
  "data": {
    "transactions": [
      {
        "transactionId": "TXN123456789",
        "amount": "1000.00",
        "type": "DEBIT",
        "description": "Transfer to UBA",
        "date": "2024-01-15T10:30:00Z",
        "status": "COMPLETED",
        "recipientName": "Jane Smith",
        "recipientAccount": "1234567890",
        "recipientBank": "UBA"
      }
    ],
    "totalCount": 25,
    "totalPages": 1
  }
}
```

**Notes:**
- Date format must be YYYY-MM-DD
- Maximum 100 items per request
- Automatically filters by user's account

### Bank Management

**Endpoint:** `POST /api/v1/wallet/get_banks`

**Purpose:** Retrieve list of Nigerian banks for transfers

**Request:**
```json
{
  "merchantID": "MERCHANT_ID_HERE"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Banks retrieved successfully",
  "responseCode": "200",
  "data": {
    "banks": [
      {
        "bankCode": "033",
        "bankName": "UNITED BANK FOR AFRICA",
        "shortName": "UBA"
      },
      {
        "bankCode": "070",
        "bankName": "FIDELITY BANK",
        "shortName": "FIDELITY"
      }
    ]
  }
}
```

**Verify Recipient Account:**

**Endpoint:** `POST /api/v1/wallet/other_bank_enquiry`

**Purpose:** Verify recipient account details before transfer

**Request:**
```json
{
  "customer": {
    "account": {
      "number": "1234567890",
      "bank": "033"
    }
  }
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Account verified successfully",
  "responseCode": "200",
  "data": {
    "accountName": "JANE DOE",
    "accountNumber": "1234567890",
    "bankName": "UNITED BANK FOR AFRICA",
    "bankCode": "033"
  }
}
```

---

## 3. Transfer Operations

### Internal Transfers (Dove-to-Dove)

#### Account-Based Internal Transfers

**Debit Transfer:**
**Endpoint:** `POST /api/v1/wallet/debit/transfer`

**Purpose:** Debit money from current user's wallet (for internal transfers)

**Request:**
```json
{
  "totalAmount": 1000.00,
  "narration": "Payment for services",
  "merchant": {
    "isFee": false,
    "merchantFeeAccount": "",
    "merchantFeeAmount": ""
  }
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Debit successful",
  "responseCode": "200",
  "data": {
    "transactionId": "WAAS202401151030001234",
    "amount": "1000.00",
    "status": "COMPLETED",
    "reference": "REF123456789"
  }
}
```

**Credit Transfer:**
**Endpoint:** `POST /api/v1/wallet/credit/transfer`

**Purpose:** Credit money to another Dove user's wallet

**Request:**
```json
{
  "accountNo": "2234567891",
  "totalAmount": 1000.00,
  "narration": "Payment received",
  "merchant": {
    "isFee": false,
    "merchantFeeAccount": "",
    "merchantFeeAmount": ""
  }
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Credit successful",
  "responseCode": "200",
  "data": {
    "transactionId": "WAAS202401151030001235",
    "amount": "1000.00",
    "status": "COMPLETED",
    "recipientAccount": "2234567891"
  }
}
```

**Notes:**
- **No fees for internal transfers** (Dove-to-Dove)
- Requires recipient's Dove account number
- Both debit and credit needed for complete transfer
- Real-time processing

#### Username-Based Internal Transfers

**Endpoint:** `POST /api/v1/wallet/internal_transfer`

**Purpose:** Transfer money to other Dove users using their username instead of account number

**Request:**
```json
{
  "recipient": {
    "username": "@uguaustine"              // Username with or without @ prefix
  },
  "order": {
    "amount": "1000.00",                   // Transfer amount
    "currency": "NGN",                     // Always NGN for Naira wallet
    "description": "Payment for lunch",    // Transfer description
    "country": "NG"                        // Always NG for Nigeria
  },
  "merchant": {
    "isFee": false,                        // No fees for internal transfers
    "merchantFeeAmount": "0",
    "merchantFeeAccount": null
  },
  "narration": "Payment for lunch"         // Custom narration
}
```

**Success Response (200):**
```json
{
  "success": true,
  "responseCode": "200",
  "message": "Transfer to @uguaustine completed successfully",
  "data": {
    "transaction_reference": "INTERNAL20250711135824561",
    "debit_reference": "INT_DEBIT20250711135824562",
    "credit_reference": "INT_CREDIT20250711135824563",
    "amount": "1000.00",
    "currency": "NGN",
    "recipient": {
      "username": "@uguaustine",
      "display_name": "Augustine Ugonna",
      "account_number": "2234567890"
    },
    "sender": {
      "account_number": "2234567891",
      "account_name": "John Doe",
      "username": "@johndoe"
    },
    "narration": "Payment for lunch",
    "transfer_type": "internal",
    "transfer_method": "debit_credit",
    "status": "successful",
    "transaction_time": "2025-07-11T13:58:24.123456",
    "processing_time_seconds": 2.45,
    "debit_successful": true,
    "credit_successful": true
  }
}
```

**Error Responses:**
- **404:** `{"success": false, "message": "User with username '@uguaustine' not found", "responseCode": "404"}`
- **400:** `{"success": false, "message": "You cannot transfer money to yourself", "responseCode": "400"}`
- **400:** `{"success": false, "message": "User '@uguaustine' does not have an active wallet account", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Invalid username format", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to debit sender account: Insufficient balance", "responseCode": "500"}`
- **500:** `{"success": false, "message": "Transfer partially failed: Money debited but credit failed. Reference: INTERNAL123. Contact support immediately.", "responseCode": "500"}`

**Notes:**
- **No fees for username-based internal transfers**
- Username format: 3-20 characters, letters/numbers/underscore only
- Can use `@uguaustine` or `uguaustine` (@ prefix optional)
- **Two-phase transfer:** First debits sender, then credits recipient
- **Critical error handling:** If debit succeeds but credit fails, requires manual intervention
- Automatic beneficiary usage tracking if recipient is saved
- Returns separate transaction references for debit and credit operations
- Real-time processing with comprehensive logging

### Username Lookup & Validation

**Endpoint:** `POST /api/v1/wallet/lookup_username`

**Purpose:** Validate username and get basic recipient information before transfer

**Query Parameters:**
```
username=@uguaustine                      // Username to lookup
```

**Success Response (200):**
```json
{
  "success": true,
  "responseCode": "200", 
  "message": "User '@uguaustine' found successfully",
  "data": {
    "username": "@uguaustine",
    "display_name": "Augustine Ugonna",
    "bank_name": "9 PAYMENT SERVICE BANK"
    // Note: Account number and sensitive data not included for privacy
  }
}
```

**Error Responses:**
- **404:** `{"success": false, "message": "User with username '@uguaustine' not found", "responseCode": "404"}`
- **400:** `{"success": false, "message": "Invalid username format", "responseCode": "400"}`

**Notes:**
- Use this endpoint to validate usernames before showing transfer UI
- Returns only public information (no account numbers or email)
- Helps prevent failed transfers due to invalid usernames
- Username format validated server-side

### External Transfers (Other Banks)

**Endpoint:** `POST /api/v1/wallet/wallet_other_banks`

**Purpose:** Transfer money to other Nigerian banks (UBA, Fidelity, etc.)

**Request:**
```json
{
  "transaction": {
    "reference": ""
  },
  "order": {
    "amount": "1000.00",
    "currency": "NGN",
    "description": "Transfer to UBA",
    "country": "Nigeria"
  },
  "customer": {
    "account": {
      "number": "1234567890",
      "bank": "033",
      "name": "JANE DOE"
    }
  },
  "merchant": {
    "isFee": true,
    "merchantFeeAmount": "20",
    "merchantFeeAccount": ""
  },
  "narration": "Payment for goods"
}
```

**Query Parameters (Optional):**
- `action` - Set to "save" to save recipient for future transfers
- `bank_name` - Bank name for recipient saving (e.g., "UNITED BANK FOR AFRICA")

**Success Response (200):**
```json
{
  "success": true,
  "message": "Transfer successful",
  "responseCode": "200",
  "data": {
    "transactionId": "TXN123456789",
    "amount": "1000.00",
    "fee": "20.00",
    "totalDebit": "1020.00",
    "status": "COMPLETED",
    "recipientName": "JANE DOE",
    "recipientAccount": "1234567890",
    "recipientBank": "UNITED BANK FOR AFRICA",
    "reference": "REF123456789"
  }
}
```

**Notes:**
- **Fixed fee of ₦20** for all external transfers
- Always verify account first with `/other_bank_enquiry`
- Recipient automatically saved if `action=save` parameter provided
- Email notification sent automatically
- Transaction saved to user's history

### Transaction Status & Tracking

**Endpoint:** `POST /api/v1/wallet/wallet_requery`

**Purpose:** Check the status of a specific transaction

**Request:**
```json
{
  "transactionId": "TXN123456789",
  "amount": 1000.00,
  "transactionType": "OTHER_BANKS",
  "transactionDate": "2024-01-15",
  "accountNo": "2234567890"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Transaction status retrieved",
  "responseCode": "200",
  "data": {
    "transactionId": "TXN123456789",
    "status": "COMPLETED",
    "amount": "1000.00",
    "statusDescription": "Transaction successful",
    "processingTime": "30 seconds"
  }
}
```

**Notes:**
- Use for tracking pending transactions
- Date format: YYYY-MM-DD
- Useful for retry mechanisms

---

## 4. Beneficiary Management

### Beneficiary System Overview

The beneficiary system allows users to save frequently used recipients for easy money transfers, with automatic usage tracking and analytics.

**Key Features:**
- **Save Recipients:** Store account details for frequent transfers
- **Usage Tracking:** Automatic statistics on transfer frequency and amounts  
- **Search & Filter:** Find beneficiaries by name, bank, or account number
- **Status Management:** Active/inactive beneficiary control
- **Analytics:** Usage statistics and insights
- **Username Support:** Special support for internal Dove users via username

**API Endpoints Summary:**
```
POST /api/v1/beneficiaries/           - Add new beneficiary
POST /api/v1/beneficiaries/username   - Add username-based internal beneficiary
GET  /api/v1/beneficiaries/           - Get user's beneficiaries (paginated)
GET  /api/v1/beneficiaries/{id}       - Get single beneficiary details
PUT  /api/v1/beneficiaries/{id}       - Update beneficiary details
DELETE /api/v1/beneficiaries/{id}     - Delete beneficiary
GET  /api/v1/beneficiaries/stats      - Get beneficiary analytics
```

### Regular Beneficiaries

#### Add Beneficiary
**Endpoint:** `POST /api/v1/beneficiaries/`

**Purpose:** Create a new beneficiary for the authenticated user

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

#### Get User Beneficiaries
**Endpoint:** `GET /api/v1/beneficiaries/`

**Purpose:** Retrieve paginated list of user's beneficiaries with optional filtering

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

#### Update Beneficiary
**Endpoint:** `PUT /api/v1/beneficiaries/{beneficiary_id}`

**Purpose:** Update beneficiary details (name, nickname, notes, status only)

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
    // Updated beneficiary object
  }
}
```

**Notes:**
- Account number and bank details cannot be modified
- Only provided fields will be updated
- Name will be auto-formatted to title case
- Updated timestamp is automatically set

#### Delete Beneficiary
**Endpoint:** `DELETE /api/v1/beneficiaries/{beneficiary_id}`

**Purpose:** Permanently delete a beneficiary

**Success Response (200):**
```json
{
  "success": true,
  "responseCode": "200",
  "message": "Beneficiary deleted successfully"
}
```

**Notes:**
- Deletion is permanent and cannot be undone
- Only the beneficiary owner can delete their beneficiaries
- All usage statistics are lost upon deletion

### Username-Based Internal Beneficiaries

#### Add Username Beneficiary
**Endpoint:** `POST /api/v1/beneficiaries/username`

**Purpose:** Add a username-based internal beneficiary for easy Dove-to-Dove transfers

**Request:**
```json
{
  "username": "uguaustine",               // Username without @ prefix
  "nickname": "Augustine",               // Optional: Display nickname
  "notes": "My friend from university"   // Optional: Personal notes
}
```

**Success Response (200):**
```json
{
  "success": true,
  "responseCode": "200",
  "message": "Username beneficiary @uguaustine added successfully",
  "data": {
    "id": "beneficiary_id_123",
    "name": "Augustine Ugonna",
    "account_number": "2234567890",
    "bank_name": "9 PAYMENT SERVICE BANK",
    "bank_code": "120001",
    "nickname": "@uguaustine",
    "notes": "Internal user: @uguaustine",
    "beneficiary_type": "INTERNAL",
    "status": "ACTIVE",
    "username": "@uguaustine"
  }
}
```

**Notes:**
- Automatically looks up username to get account details
- Creates regular beneficiary with INTERNAL type
- Username validation ensures user exists and has active wallet
- Integrates seamlessly with transfer system

### Beneficiary Analytics & Statistics

**Endpoint:** `GET /api/v1/beneficiaries/stats`

**Purpose:** Retrieve user's beneficiary analytics and statistics

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

**Notes:**
- Most used beneficiary is determined by total transfer count
- Recent beneficiaries are sorted by last usage date
- Statistics include both active and inactive beneficiaries
- Useful for analytics dashboards and insights

#### Automatic Usage Tracking

When transfers are made to saved beneficiaries:
- **Usage statistics** are automatically updated
- **Transfer frequency** and total amounts tracked
- **Most-used** and recently-used beneficiaries identified
- **Seamless integration** with both account-based and username-based transfers

**Benefits:**
- **Quick Access:** Select from saved recipients instead of typing each time
- **Usage Insights:** See transfer patterns and frequently contacted users
- **Easy Management:** Update nicknames and notes for better organization
- **Seamless Integration:** All transfer types automatically update beneficiary stats

---

## 5. Integration Guidelines

### Complete Transfer Workflows

#### External Transfer Flow:
```
1. GET /wallet/ → Check balance
2. POST /get_banks → Get bank list
3. POST /other_bank_enquiry → Verify recipient account
4. POST /wallet_other_banks → Execute transfer
5. POST /wallet_requery → Confirm status (if needed)
```

#### Internal Transfer Flow (Account-Based):
```
1. GET /wallet/ → Check sender balance
2. POST /debit/transfer → Debit sender
3. POST /credit/transfer → Credit recipient
4. GET /wallet/ → Confirm updated balance
```

#### Username-Based Internal Transfer Flow:
```
1. GET /wallet/ → Check sender balance
2. POST /lookup_username → Validate recipient username (optional)
3. POST /internal_transfer → Execute username-based transfer
   - Automatically debits sender and credits recipient
   - Returns both debit and credit transaction references
4. GET /wallet/ → Confirm updated balance
```

#### Transfer with Beneficiary Integration:
```
1. GET /beneficiaries/ → Load saved beneficiaries
2. User selects beneficiary OR enters new recipient details
3. If new recipient: Optional POST /beneficiaries/ to save
4. Execute transfer using appropriate endpoint
5. Usage statistics automatically updated for saved beneficiaries
```

### Fee Structure

#### Internal Transfers (Dove-to-Dove):
- **Fee:** ₦0 (Free)
- **Processing:** Real-time
- **Limits:** Based on KYC tier
- **Both account-based and username-based transfers are free**

#### External Transfers (Other Banks):
- **Fee:** ₦20 (Fixed)
- **Processing:** Usually instant, up to 30 seconds
- **Limits:** Based on KYC tier
- **Supported Banks:** All Nigerian banks via 9PSB

### Error Handling Guide

#### Common Error Scenarios:

**Insufficient Balance:**
```json
{
  "success": false,
  "message": "Insufficient wallet balance",
  "responseCode": "201"
}
```

**Invalid Account:**
```json
{
  "success": false,
  "message": "Invalid recipient account number",
  "responseCode": "201"
}
```

**Username Not Found:**
```json
{
  "success": false,
  "message": "User with username '@username' not found",
  "responseCode": "404"
}
```

**Network Issues:**
```json
{
  "success": false,
  "message": "Network error communicating with banking service",
  "responseCode": "NETWORK_ERROR"
}
```

**Service Unavailable:**
```json
{
  "success": false,
  "message": "Banking service temporarily unavailable",
  "responseCode": "502"
}
```

#### Error Response Format:
All error responses follow this structure:
```json
{
  "success": false,
  "message": "Human readable error message",
  "responseCode": "ERROR_CODE",
  "data": {
    "additional_context": "..."
  }
}
```

### Security & Validation

#### Amount Validation:
- Minimum transfer: ₦10
- Maximum transfer: Based on KYC tier
- Amount format: Decimal with 2 places (e.g., 1000.00)

#### Account Validation:
- Nigerian account numbers: 10 digits
- Bank codes: 3 digits
- Always verify accounts before transfers

#### Username Validation:
- Format: 3-20 characters, letters/numbers/underscore only
- Case insensitive
- @ prefix optional but automatically handled

#### Rate Limiting:
- Account verification: 10 requests/minute
- Transfers: 5 requests/minute
- Transaction history: 20 requests/minute
- Beneficiary operations: 15 requests/minute

---

## 6. Development Resources

### Mobile App Implementation Tips

#### UX Recommendations:
1. **Balance Check:** Always check balance before transfer UI
2. **Account Verification:** Verify and show recipient name before transfer
3. **Fee Display:** Clearly show ₦20 fee for external transfers
4. **Loading States:** Show loading for transfers (can take up to 30 seconds)
5. **Retry Logic:** Implement retry for network errors
6. **Transaction History:** Cache recent transactions for offline viewing
7. **Beneficiary Quick Access:** Show most-used and recent beneficiaries prominently
8. **Username Support:** Provide clear UI for username-based transfers with @ prefix handling

#### Validation Requirements:
- **Amount:** Positive decimal, minimum ₦10
- **Account Number:** Exactly 10 digits for Nigerian banks
- **Bank Selection:** Use bank codes from `/get_banks`
- **Narration:** 1-100 characters, required
- **Username:** 3-20 characters, alphanumeric + underscore, case insensitive

### Testing & Development

#### Test Scenarios:
1. **Successful Transfer:** Valid account, sufficient balance
2. **Insufficient Balance:** Amount exceeds available balance
3. **Invalid Account:** Non-existent recipient account
4. **Invalid Username:** Non-existent or invalid format username
5. **Network Failure:** Simulate network interruption
6. **Service Downtime:** 9PSB service unavailable
7. **Beneficiary Operations:** Add, update, delete, search beneficiaries
8. **Usage Tracking:** Verify automatic beneficiary usage updates

#### Development Environment:
- **API Base URL:** Configure for dev/staging/production
- **Test Accounts:** Use test Nigerian bank accounts
- **Mock Data:** Implement mock responses for offline development

### Performance & Caching

#### Recommended Practices:
- **Cache bank list** (update weekly)
- **Cache beneficiary lists** for 5-10 minutes
- **Debounce account verification** (wait for user to finish typing)
- **Debounce username lookup** (300ms delay)
- **Implement pull-to-refresh** for transaction history
- **Show estimated transfer time** (30 seconds for external)
- **Use pagination** for large beneficiary lists
- **Show loading states** during API calls
- **Refresh cache** after add/update/delete operations

#### Expected Response Times:
- **Add Beneficiary:** 1-3 seconds
- **Get Beneficiaries:** 1-2 seconds
- **Update/Delete:** 1-2 seconds
- **Get Statistics:** 2-3 seconds
- **Username Lookup:** 1-2 seconds
- **Internal Transfer:** 2-5 seconds
- **External Transfer:** 5-30 seconds

### Monitoring & Logging

#### Client-Side Logging:
Log these events for debugging:
- Transfer initiation with amount and recipient
- Account verification results
- Username lookup attempts
- Transfer completion/failure
- Network errors and retry attempts
- Beneficiary operations (add, update, delete)

#### Server-Side Tracking:
All endpoints include comprehensive logging:
- Request/response timing
- External API call details
- Error tracking with full context
- Transaction reference tracking
- Beneficiary usage updates

---

## 7. Future Expansion

### Multi-Currency Support (Coming Soon)

This documentation structure is designed to accommodate future multi-currency wallet operations:

**Planned Features:**
- USD wallet operations
- Multi-currency transfers
- Currency exchange services
- Cross-currency beneficiaries
- Enhanced analytics across currencies

**Documentation Structure:**
```
WALLET_OPERATIONS.md
├── Naira (NGN) Operations [Current]
├── USD Operations [Coming Soon]
├── Multi-Currency Features [Coming Soon]
└── Cross-Currency Operations [Coming Soon]
```

### Support & Troubleshooting

#### Common Issues:

**"User has no nuban account yet":**
- User needs to complete wallet creation via `/kyc/create_nuban`

**"Account verification failed":**
- Check account number format (10 digits)
- Verify bank code is correct
- Account may not exist

**"Username not found":**
- Verify username spelling and format
- User may not have completed wallet setup
- Username may not exist in system

**"Transfer timeout":**
- Implement retry mechanism
- Check transaction status with `/wallet_requery`
- May need manual verification

#### Debug Information:
Include in support requests:
- Transaction ID or reference
- Error message and response code
- Amount and recipient details
- Timestamp of attempt
- Username (for username-based transfers)

---

This comprehensive guide provides complete implementation details for all wallet operations including Naira transfers, beneficiary management, and username-based internal transfers. The backend handles all complex business logic, 9PSB integration, and security - focus on creating an intuitive user experience for wallet management and transfers!