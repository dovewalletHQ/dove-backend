# Dove Miden Virtual Card API - Frontend Developer Guide

## Overview

This guide provides comprehensive documentation for frontend developers to integrate with the Dove FastAPI backend's Miden Virtual Card endpoints. The backend handles all communication with Miden's API, providing a simplified interface for frontend applications.

**Architecture Flow:**
```
Frontend App → Dove FastAPI Backend → Miden API
```

All endpoints are prefixed with `/api/v1/miden` and require user authentication through the Dove platform.

## Authentication

All Miden endpoints require Dove platform authentication. Include the logged-in user's JWT token in the Authorization header:

```javascript
headers: {
  'Authorization': `Bearer ${doveUserToken}`,  // User's Dove platform JWT
  'Content-Type': 'application/json'
}
```

The backend automatically handles Miden API authentication using stored credentials.

## Base Response Format

All endpoints return the standard Dove API response format:

```javascript
{
  "success": boolean,        // Operation success status
  "message": string,         // Human-readable message
  "data": object | null,     // Response data (contains Miden API response)
  "code": string | null,     // Error code if applicable
  "http_status": number      // HTTP status code
}
```

The `data` field contains the response from Miden's API, wrapped in our backend response structure.

## API Endpoints

### 🏢 Customer Management

#### 1. Get All Customers
```javascript
GET /api/v1/miden/customers

// Query Parameters (all optional)
const params = {
  page_number: 1,           // Page number (default: 1)
  page_size: 20,           // Items per page (default: 20, max: 100)
  customer_id: "uuid",     // Filter by customer ID
  last_name: "Doe",        // Filter by last name
  email: "user@email.com", // Filter by email
  activated: true,         // Filter by activation status
  terminated: false        // Filter by termination status
};

// Example Usage
const response = await fetch(`${DOVE_API_BASE}/api/v1/miden/customers?${new URLSearchParams(params)}`, {
  headers: { 'Authorization': `Bearer ${doveUserToken}` }
});
```

#### 2. Get Specific Customer
```javascript
GET /api/v1/miden/customers/{customer_id}

// Example
const customerId = "A1B90246-9C0B-4106-4B51-08DD550584A2";
const response = await fetch(`${DOVE_API_BASE}/api/v1/miden/customers/${customerId}`, {
  headers: { 'Authorization': `Bearer ${doveUserToken}` }
});
```

#### 3. Get Customer's Cards
```javascript
GET /api/v1/miden/customers/{customer_id}/cards

// Query Parameters (optional)
const params = {
  currency_code: "USD",    // Filter by currency (USD/NGN)
  active_only: true        // Return only active cards
};
```

---

### 💳 Card Management

#### 1. Get All Cards
```javascript
GET /api/v1/miden/cards

// Query Parameters (all optional)
const params = {
  page_number: 1,
  page_size: 20,
  card_id: "uuid",
  customer_id: "uuid",
  currency_code: "USD",    // USD or NGN
  activated: true,
  terminated: false,
  from_date: "2024-01-01", // YYYY-MM-DD
  to_date: "2024-12-31"    // YYYY-MM-DD
};
```

#### 2. Get Specific Card
```javascript
GET /api/v1/miden/cards/{card_id}

// Query Parameters
const params = {
  decrypt_details: true    // Get decrypted card number, CVV, expiry
};

// Response includes encrypted or decrypted card details
```

#### 3. Get USD Cards Only
```javascript
GET /api/v1/miden/cards/usd

// Query Parameters
const params = {
  page_number: 1,
  page_size: 20,
  customer_id: "uuid",     // Optional: filter by customer
  active_only: true        // Optional: only active cards
};
```

#### 4. Get Active Cards
```javascript
GET /api/v1/miden/cards/active

// Similar parameters to get all cards
```

---

### ⚙️ Card Actions

#### 1. Set Card PIN (NGN Cards Only)
```javascript
PATCH /api/v1/miden/cards/{card_id}/pin

// Request Body
const body = {
  pin: "1234",           // 4-digit PIN
  confirm_pin: "1234"    // Must match PIN
};

await fetch(`${DOVE_API_BASE}/api/v1/miden/cards/${cardId}/pin`, {
  method: 'PATCH',
  headers: {
    'Authorization': `Bearer ${doveUserToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(body)
});
```

#### 2. Replace Card (Decommissioned BINs)
```javascript
PATCH /api/v1/miden/cards/{card_id}/replace

// No body required - automatically transfers balance
await fetch(`${DOVE_API_BASE}/api/v1/miden/cards/${cardId}/replace`, {
  method: 'PATCH',
  headers: { 'Authorization': `Bearer ${doveUserToken}` }
});
```

#### 3. Toggle Airline Payments
```javascript
PATCH /api/v1/miden/cards/{card_id}/airline-payments?enable=true

// Query Parameters
const enable = true; // true to enable, false to disable

await fetch(`${DOVE_API_BASE}/api/v1/miden/cards/${cardId}/airline-payments?enable=${enable}`, {
  method: 'PATCH',
  headers: { 'Authorization': `Bearer ${doveUserToken}` }
});
```

---

### 🚀 Card Issuance

#### 1. Issue Lite Card (Non-reloadable)
```javascript
POST /api/v1/miden/cards/issue-lite

// Request Body
const body = {
  first_name: "John",
  last_name: "Doe",
  phone: "+2348012345678",
  address1: "123 Main Street",
  address2: "Apt 4B",          // Optional
  city: "Lagos",
  state: "Lagos",
  zipcode: "100001",
  country: "NG",
  payee: "John Doe",
  swipe_count: 5,              // Number of uses allowed
  card_balance: 100.00,        // USD amount
  expiration_date: "2025-12-31", // Optional (YYYY-MM-DD)
  terminate_date: "2025-12-31",  // Optional (YYYY-MM-DD)
  customer_id: "CUST_12345",
  card_brand: "Visa",          // "Visa" or "Mastercard"
  wallet_currency: "USD"       // "USD" or "NGN"
};
```

#### 2. Re-issue Card (Additional Cards)
```javascript
POST /api/v1/miden/cards/reissue

// Request Body
const body = {
  card_customer_id: "uuid",
  initial_balance: 50.00,      // Not used if shared_balance is true
  card_brand: "Visa",
  name_on_card: "John Doe",
  client_reference: "REF_12345",
  wallet_currency: "USD",
  
  // Optional fields
  address: {                   // Optional - uses first card address if not provided
    address_line1: "123 Main St",
    address_line2: "Apt 4B",
    city: "Lagos",
    state: "Lagos",
    zipcode: "100001",
    country: "NG"
  },
  
  shared_balance: false,       // Share balance with another card
  card_id: "uuid",            // Required if shared_balance is true
  cardless_payment: false,     // Enable cardless transactions
  
  card_limits: {              // Optional spending limits
    daily_limit: 1000.00,
    transaction_limit: 500.00
  },
  
  white_listed_mccs: ["5411", "5812"], // Allowed merchant categories
  black_listed_mccs: ["7995", "4411"]  // Blocked merchant categories
};
```

---

### 💰 Transaction Management

#### 1. Get Transaction History
```javascript
GET /api/v1/miden/transactions

// Query Parameters (all optional)
const params = {
  card_id: "uuid",
  transaction_type: "Authorization", // Authorization, TopUp, Withdrawal, etc.
  transaction_status: "Pending",     // Pending, Success, Failed
  start_date: "2024-01-01",         // YYYY-MM-DD
  end_date: "2024-01-31",           // YYYY-MM-DD
  client_reference: "REF_123",
  page_number: 1,
  page_size: 20
};
```

#### 2. Get Transaction Status
```javascript
GET /api/v1/miden/transactions/status/{reference}

// Example
const reference = "17243350301724335030";
const response = await fetch(`${DOVE_API_BASE}/api/v1/miden/transactions/status/${reference}`, {
  headers: { 'Authorization': `Bearer ${doveUserToken}` }
});
```

#### 3. Top Up Card
```javascript
PATCH /api/v1/miden/cards/{card_id}/topup

// Request Body
const body = {
  card_id: "uuid",
  amount: 100.00,              // Amount to add
  client_reference: "TOP_123", // Unique reference
  wallet_currency: "USD"       // Source wallet currency
};
```

#### 4. Withdraw from Card
```javascript
PATCH /api/v1/miden/cards/{card_id}/withdraw

// Request Body
const body = {
  card_id: "uuid",
  amount: 50.00,               // Amount to withdraw
  reason: "Unused funds",     // Reason for withdrawal
  client_reference: "WD_123"  // Unique reference
};

// Note: Minimum $1 balance must remain on card
```

#### 5. Reprocess Cross-border Charges
```javascript
PATCH /api/v1/miden/cards/{card_id}/reprocess-cross-border

// No body required - retries failed international transactions
await fetch(`${DOVE_API_BASE}/api/v1/miden/cards/${cardId}/reprocess-cross-border`, {
  method: 'PATCH',
  headers: { 'Authorization': `Bearer ${doveUserToken}` }
});
```

#### 6. Card-to-Card Transfer (Coming Soon)
```javascript
PATCH /api/v1/miden/cards/transfer

// Request Body
const body = {
  card_id: "source-card-uuid",
  amount: 25.00,
  beneficiary_card_id: "destination-card-uuid",
  reason: "Team budget allocation"
};
```

---

### 💱 Exchange Rates

#### Get Exchange Rate
```javascript
GET /api/v1/miden/rates/{source_currency}

// Example: Get NGN to USD rate
const response = await fetch('/api/v1/miden/rates/NGN', {
  headers: { 'Authorization': `Bearer ${token}` }
});

// Response includes buy and sell rates
const data = response.data;
console.log(`Buy Rate: ${data.buyRate}`);
console.log(`Sell Rate: ${data.sellRate}`);
```

---

## Frontend Implementation Examples

### React Hook for Card Management
```javascript
import { useState, useEffect } from 'react';

export const useCustomerCards = (customerId) => {
  const [cards, setCards] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchCards = async () => {
      try {
        const response = await fetch(`${DOVE_API_BASE}/api/v1/miden/customers/${customerId}/cards`, {
          headers: { 'Authorization': `Bearer ${getDoveToken()}` }
        });
        const result = await response.json();
        
        if (result.success) {
          setCards(result.data.cards);
        } else {
          setError(result.message);
        }
      } catch (err) {
        setError('Failed to fetch cards');
      } finally {
        setLoading(false);
      }
    };

    if (customerId) {
      fetchCards();
    }
  }, [customerId]);

  return { cards, loading, error };
};
```

### Card Issuance Form Handler
```javascript
const handleIssueLiteCard = async (formData) => {
  try {
    const response = await fetch(`${DOVE_API_BASE}/api/v1/miden/cards/issue-lite`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${getDoveToken()}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        first_name: formData.firstName,
        last_name: formData.lastName,
        phone: formData.phone,
        address1: formData.address,
        city: formData.city,
        state: formData.state,
        zipcode: formData.zipcode,
        country: formData.country,
        payee: `${formData.firstName} ${formData.lastName}`,
        swipe_count: parseInt(formData.swipeCount),
        card_balance: parseFloat(formData.balance),
        customer_id: formData.customerId,
        card_brand: formData.cardBrand,
        wallet_currency: "USD"
      })
    });

    const result = await response.json();
    
    if (result.success) {
      // Card issued successfully
      const cardData = result.data.data;
      console.log('New card ID:', cardData.cardId);
      console.log('Card number:', cardData.cardNumber);
      return cardData;
    } else {
      throw new Error(result.message);
    }
  } catch (error) {
    console.error('Card issuance failed:', error);
    throw error;
  }
};
```

### Transaction History Component
```javascript
const TransactionHistory = ({ cardId }) => {
  const [transactions, setTransactions] = useState([]);
  const [pagination, setPagination] = useState({
    currentPage: 1,
    pageSize: 20,
    totalPages: 0
  });

  const fetchTransactions = async (page = 1) => {
    const params = new URLSearchParams({
      card_id: cardId,
      page_number: page,
      page_size: pagination.pageSize
    });

    const response = await fetch(`${DOVE_API_BASE}/api/v1/miden/transactions?${params}`, {
      headers: { 'Authorization': `Bearer ${getDoveToken()}` }
    });

    const result = await response.json();
    
    if (result.success) {
      setTransactions(result.data.transactions || []);
      setPagination({
        currentPage: result.data.currentPage,
        pageSize: result.data.pageSize,
        totalPages: result.data.totalPages
      });
    }
  };

  useEffect(() => {
    if (cardId) {
      fetchTransactions();
    }
  }, [cardId]);

  return (
    <div>
      {transactions.map(transaction => (
        <div key={transaction.id}>
          <p>Amount: ${transaction.amount}</p>
          <p>Type: {transaction.type}</p>
          <p>Status: {transaction.status}</p>
          <p>Date: {new Date(transaction.createdAt).toLocaleDateString()}</p>
        </div>
      ))}
      
      {/* Pagination controls */}
      <div>
        <button 
          onClick={() => fetchTransactions(pagination.currentPage - 1)}
          disabled={pagination.currentPage === 1}
        >
          Previous
        </button>
        <span>Page {pagination.currentPage} of {pagination.totalPages}</span>
        <button 
          onClick={() => fetchTransactions(pagination.currentPage + 1)}
          disabled={pagination.currentPage === pagination.totalPages}
        >
          Next
        </button>
      </div>
    </div>
  );
};
```

## Error Handling

Always check the `success` field in responses:

```javascript
const response = await fetch(`${DOVE_API_BASE}/api/v1/miden/cards`, {
  headers: { 'Authorization': `Bearer ${doveUserToken}` }
});

const result = await response.json();

if (result.success) {
  // Handle successful response
  const cards = result.data.cards;
} else {
  // Handle error
  console.error('API Error:', result.message);
  // Show user-friendly error message
  showErrorMessage(result.message);
}
```

## Common Response Codes

- `200` - Success
- `201` - Created (for card issuance)
- `400` - Bad Request (validation errors)
- `401` - Unauthorized (invalid token)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found (card/customer doesn't exist)
- `500` - Internal Server Error

## Security Considerations

1. **Never log sensitive data** (card numbers, CVVs, PINs)
2. **Always use HTTPS** for API calls
3. **Validate user input** before sending to API
4. **Handle tokens securely** (use httpOnly cookies or secure storage)
5. **Implement rate limiting** on the frontend to prevent abuse

## Testing

- Development: `https://mydove-fastapi-backend-pr-26.onrender.com`
- Staging: `https://mydove-fastapi-backend-pr-26.onrender.com`
- Production: `https://mydove-fastapi-backend-j1ki.onrender.com`

The backend handles Miden sandbox/production switching based on environment configuration. All test transactions are safe in development/staging environments.

---

## Quick Start Checklist

1. ✅ Set up authentication headers
2. ✅ Implement error handling
3. ✅ Test with sandbox credentials
4. ✅ Create card issuance flow
5. ✅ Add transaction history
6. ✅ Implement card controls (freeze/unfreeze)
7. ✅ Add balance management (top-up/withdrawal)
8. ✅ Test all user flows thoroughly

## Environment Variables

Set up your frontend environment variables:

```javascript
// .env.development
REACT_APP_DOVE_API_BASE=http://localhost:8000
// or your local backend URL

// .env.production  
REACT_APP_DOVE_API_BASE=https://api.dove.com
```

```javascript
// Use in your code
const DOVE_API_BASE = process.env.REACT_APP_DOVE_API_BASE;
```

For additional support, refer to the main MIDEN.md business documentation or contact the backend development team.