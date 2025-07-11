# Dove Mobile App Registration API Guide - Frontend Developers

## Overview for Frontend Developers

This guide provides the essential API information needed to implement the Dove app registration flow in React Native. No code examples - just the API specifications, data flow, and integration requirements.

---

## Registration Flow Overview

### User Journey:
```
1. Email + Password Registration → 2. OTP Verification & Account Creation → 3. Set Passcode → 4. Identity Verification (BVN + DOB) → 5. Phone Number Collection → 6. Set Username → 7. Set Transaction PIN → 8. Complete Profile → 9. Create Wallet
```

### API Endpoints Summary:
```
POST /api/v1/auth/register               - Send 5-digit OTP to email (with password)
POST /api/v1/auth/verify-otp             - Verify OTP & create account with email/password
POST /api/v1/auth/set-passcode           - Set 6-digit passcode
POST /api/v1/auth/identity-verification  - Verify identity with BVN + date of birth
POST /api/v1/auth/set-phone-number       - Set phone number (no verification required)
POST /api/v1/auth/set-username           - Set unique username
POST /api/v1/auth/set-transaction-pin    - Set 4-digit transaction PIN
POST /api/v1/user/update-profile         - Complete profile with all info
POST /api/v1/kyc/create_nuban            - Create wallet (uses stored profile data)
```

### **Flow Separation:**

**Authentication Flow** (Email → Password → Verify → Create Account → Passcode):
- Steps 1-3: Basic account creation with minimal data
- User gets authenticated access token after Step 2

**Identity Verification Flow** (BVN → DOB → Phone → Username → PIN):
- Steps 4-7: Progressive data collection while authenticated
- All steps require authentication token from Step 2

**Profile & Wallet Flow** (Complete Profile → Create Wallet):
- Steps 8-9: Final profile completion and financial account setup
- Unchanged from previous implementation

---

## API Endpoints Documentation

### Step 1: Email + Password Registration
**Endpoint:** `POST /api/v1/auth/register`

**Purpose:** Send 5-digit OTP verification code to user's email (password required)

**Request:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "OTP sent successfully",
  "responseCode": "200",
  "data": {
    "email": "user@example.com",
    "otp": "12345"  // 5-digit OTP, only in development environment
  }
}
```

**Error Responses:**
- **400:** `{"success": false, "message": "User already exists", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Registration failed", "responseCode": "500"}`

**Notes:**
- Email is automatically converted to lowercase
- Password is stored securely with the OTP for account creation
- OTP is now 5 digits instead of 6
- OTP expires after 5 minutes (configurable)
- OTP is only returned in development mode for testing

---

### Step 2: OTP Verification & Account Creation
**Endpoint:** `POST /api/v1/auth/verify-otp`

**Purpose:** Verify 5-digit OTP and create user account with email/password only

**Request:**
```json
{
  "email": "user@example.com",
  "otp": "12345"
}
```

**Success Response (200):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 3600,
  "user": "{\"id\":\"...\",\"email\":\"user@example.com\",\"status\":\"ACTIVE\",...}"
}
```

**Error Responses:**
- **400:** `{"success": false, "message": "Invalid or expired OTP", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Email already registered", "responseCode": "400"}`
- **500:** `{"success": false, "message": "User creation failed", "responseCode": "500"}`

**Notes:**
- This endpoint creates the user account AND returns authentication token
- Store the `access_token` securely for subsequent API calls
- Password from Step 1 is hashed automatically by the backend
- User status is set to "ACTIVE" after successful verification
- Account is created with minimal data (email + password only)
- Use this token for all subsequent registration steps

---

### Step 3: Set Passcode
**Endpoint:** `POST /api/v1/auth/set-passcode`

**Purpose:** Set up a 6-digit passcode for security

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request:**
```json
{
  "passcode": "123456"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Passcode set successfully",
  "responseCode": "200"
}
```

**Error Responses:**
- **401:** `{"success": false, "message": "Unauthorized", "responseCode": "401"}`
- **400:** `{"success": false, "message": "Invalid passcode format", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to set passcode", "responseCode": "500"}`

**Notes:**
- Passcode must be exactly 6 digits (000000-999999)
- Passcode is automatically hashed before storage
- Required for app security and authentication
- User must be authenticated (logged in) to set passcode

---

### Step 4: Identity Verification
**Endpoint:** `POST /api/v1/auth/identity-verification`

**Purpose:** Verify user identity with BVN and date of birth

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request:**
```json
{
  "bvn": "12345678901",
  "date_of_birth": "1990-01-15"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Identity verification successful",
  "responseCode": "200"
}
```

**Error Responses:**
- **401:** `{"success": false, "message": "Unauthorized", "responseCode": "401"}`
- **400:** `{"success": false, "message": "Invalid BVN format", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Invalid date format. Use YYYY-MM-DD", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to verify identity", "responseCode": "500"}`

**Notes:**
- BVN must be exactly 11 digits (numeric only)
- Date of birth format must be YYYY-MM-DD
- BVN is encrypted before storage for security
- Required for compliance and identity verification

---

### Step 5: Set Phone Number
**Endpoint:** `POST /api/v1/auth/set-phone-number`

**Purpose:** Set phone number (no verification required at this step)

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request:**
```json
{
  "phone_number": "+2348123456789"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Phone number set successfully",
  "responseCode": "200",
  "data": {
    "phone_number": "+2348123456789"
  }
}
```

**Error Responses:**
- **401:** `{"success": false, "message": "Unauthorized", "responseCode": "401"}`
- **400:** `{"success": false, "message": "Invalid phone number format", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to set phone number", "responseCode": "500"}`

**Notes:**
- Phone number must be in international format (+CountryCodeNumber)
- No OTP verification required at this step
- Phone can be verified later by user in their own time
- Phone number is stored in both user.phone and user.kyc.phone_2

---

### Step 6: Set Username
**Endpoint:** `POST /api/v1/auth/set-username`

**Purpose:** Set unique username for the user

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request:**
```json
{
  "username": "johnDoe123"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Username set successfully",
  "responseCode": "200",
  "data": {
    "username": "johnDoe123"
  }
}
```

**Error Responses:**
- **401:** `{"success": false, "message": "Unauthorized", "responseCode": "401"}`
- **400:** `{"success": false, "message": "Username is already taken", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Invalid username format", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to set username", "responseCode": "500"}`

**Notes:**
- Username must be 3-20 characters, alphanumeric + underscore only
- Username must be unique across all users
- Case-sensitive validation
- Can be used for login along with email

---

### Step 7: Set Transaction PIN
**Endpoint:** `POST /api/v1/auth/set-transaction-pin`

**Purpose:** Set up a 4-digit transaction PIN for financial operations during onboarding

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request:**
```json
{
  "transaction_pin": "1234"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Transaction PIN set successfully",
  "responseCode": "200"
}
```

**Error Responses:**
- **401:** `{"success": false, "message": "Unauthorized", "responseCode": "401"}`
- **400:** `{"success": false, "message": "Invalid PIN format", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to set transaction PIN", "responseCode": "500"}`

**Notes:**
- Transaction PIN must be exactly 4 digits (0000-9999)
- PIN is automatically hashed before storage
- Required for financial operations like transfers and payments
- User must be authenticated (logged in) to set PIN
- PIN can be updated later through user settings

---

### Step 8: Complete Profile
**Endpoint:** `POST /api/v1/user/update-profile`

**Purpose:** Complete user profile with remaining information needed for wallet creation

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request (Flexible - All Fields Optional):**
```json
{
  "first_name": "John",
  "middle_name": "William",
  "last_name": "Doe",
  "phone": "+2348123456789",
  "gender": "male",
  "date_of_birth": "1990-01-15",
  "bvn": "12345678901",
  "address_line_1": "123 Main Street",
  "city": "Lagos",
  "state": "Lagos"
}
```

**Request Examples (Partial Updates):**

*Only update missing name fields:*
```json
{
  "first_name": "John",
  "last_name": "Doe",
  "gender": "male"
}
```

*Only update address:*
```json
{
  "address_line_1": "123 Main Street",
  "city": "Lagos",
  "state": "Lagos"
}
```

*Update single field:*
```json
{
  "gender": "male"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Profile updated successfully",
  "responseCode": "200",
  "data": {
    "id": "user_id_string"
  }
}
```

**Alternative Success Response (No Changes):**
```json
{
  "success": true,
  "message": "No updates necessary - all provided data already exists",
  "responseCode": "200",
  "data": {
    "id": "user_id_string"
  }
}
```

**Error Responses:**
- **401:** `{"success": false, "message": "Unauthorized", "responseCode": "401"}`
- **400:** `{"success": false, "message": "At least one field must be provided for update", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Invalid date format. Use YYYY-MM-DD", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to update profile", "responseCode": "500"}`

**Notes:**
- **Required for wallet creation** - all necessary fields must be completed eventually
- **Flexible to prevent API breaking** - fields are optional so API won't break if users don't re-submit data already provided during registration
- **ALL FIELDS STILL REQUIRED FOR WALLET** - complete profile needed for wallet creation
- **Data from registration** - BVN, DOB, and phone are already collected in Steps 4-5
- **Send only missing data** - typically just names, gender, and address after registration
- **Smart duplicate detection** - won't update fields with identical values
- **BVN comparison** - decrypts existing BVN to avoid unnecessary updates
- **Partial address updates** - can update individual address fields
- **No breaking changes** - endpoint handles any combination of fields gracefully
- BVN is encrypted before storage
- Phone number is saved in both `user.phone` and `user.kyc.phone_2`
- Date format must be YYYY-MM-DD
- Country is automatically set to Nigeria

---

### Step 9: Create Wallet
**Endpoint:** `POST /api/v1/kyc/create_nuban`

**Purpose:** Create virtual bank account using stored profile information

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request:**
```json
{}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "NUBAN account created successfully",
  "responseCode": "200",
  "data": {
    "account_number": "2234567890",
    "account_name": "John William Doe",
    "bank_name": "9 PAYMENT SERVICE BANK",
    "customer_id": "CUST123456"
  }
}
```

**Error Responses:**
- **401:** `{"success": false, "message": "Unauthorized", "responseCode": "401"}`
- **400:** `{"success": false, "message": "Please complete your profile first before creating a wallet", "responseCode": "400"}`
- **502:** `{"success": false, "message": "Banking service unavailable", "responseCode": "502"}`

**Notes:**
- Uses profile information stored in Step 5 to create wallet
- No additional data required in request body
- User must complete profile (Step 5) before wallet creation
- Creates real virtual bank account via 9PSB API
- Account number can be used to receive payments
- Process may take a few seconds due to external API call
- BVN and address from profile are used for 9PSB requirements

---

## Authentication & Security

### **Token Management:**
- **Token Type:** JWT Bearer token
- **Expiration:** 60 minutes (3600 seconds)
- **Storage:** Store securely in device keychain/secure storage
- **Usage:** Include in `Authorization` header for protected endpoints

### **Credential Management:**
- **Password:** 8+ characters for login authentication
- **Passcode:** 6-digit security code (set during registration)
- **Transaction PIN:** 4-digit code for financial operations (set during onboarding)

### **Token Format:**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### **Session Management:**
- User session is cached on backend for performance
- Token automatically refreshed on successful API calls
- Logout by deleting token from device storage

---

## User Account States

### **Registration States:**
```
1. Email Submitted → PENDING (OTP sent)
2. OTP Verified → ACTIVE (account created with password + passcode)
3. Phone Verified → ACTIVE (phone number confirmed)
4. Transaction PIN Set → ACTIVE (financial security enabled)
5. Profile Completed → ACTIVE (all info stored, ready for wallet)
6. Wallet Created → ACTIVE (full financial access)
```

### **Account Levels:**
- **KYC Level:** LEVEL_1 (default) → LEVEL_2 → LEVEL_3
- **Account Tier:** TIER_1 (default) → TIER_2 → TIER_3
- **Status:** PENDING → ACTIVE

---

## Error Handling Guide

### **Common Error Scenarios:**

**Network Errors:**
- Handle timeout errors gracefully
- Show retry options for failed requests
- Display offline state when network unavailable

**Validation Errors:**
- **Invalid Email:** Show proper email format message
- **Weak Password:** Display password requirements (8+ characters)
- **Invalid Passcode:** Show passcode format error (must be 6 digits)
- **Invalid OTP:** Clear OTP input and show error
- **Expired OTP:** Show resend option

**Server Errors:**
- **400 Bad Request:** Show specific validation messages
- **401 Unauthorized:** Redirect to login screen
- **500 Server Error:** Show "try again later" message
- **502 Service Unavailable:** Show service maintenance message

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

## Timing & Performance

### **Expected Response Times:**
- **Email Registration:** 1-3 seconds
- **OTP Verification:** 2-5 seconds
- **Profile Update:** 1-2 seconds  
- **Wallet Creation:** 5-10 seconds (external banking API)

### **Timeout Recommendations:**
- Set request timeout to 15 seconds for wallet creation
- Set request timeout to 10 seconds for other endpoints
- Show loading states for requests over 2 seconds

---

## Additional Authentication Endpoints

### **Password Reset Flow:**

**1. Request Reset OTP:**
```
POST /api/v1/auth/send-reset-password-otp
Body: {"email": "user@example.com"}
```

**2. Verify Reset OTP:**
```
POST /api/v1/auth/verify-reset-password-otp  
Body: {"email": "user@example.com", "otp": "123456"}
```

**3. Reset Password:**
```
POST /api/v1/auth/reset-password
Body: {"email": "user@example.com", "new_password": "newPassword123", "otp": "123456"}
```

### **Login Endpoint:**
```
POST /api/v1/auth/login
Body: {"username": "user@example.com", "password": "userPassword"}
```

---

## Mobile App Considerations

### **Data Storage:**
- Store access token securely (Keychain/Keystore)
- Cache user data for offline access
- Store wallet information locally after creation

### **User Experience:**
- Auto-focus next OTP input field
- Show password strength indicator
- Implement auto-resend OTP countdown
- Display clear loading states during API calls

### **Validation Requirements:**
- **Email:** Valid email format
- **Password:** Minimum 8 characters (backend enforced)
- **OTP:** Exactly 5 digits (email verification only)
- **Passcode:** Exactly 6 digits (numeric only)
- **BVN:** Exactly 11 digits (numeric only)
- **Date of Birth:** YYYY-MM-DD format
- **Phone Number:** International format (+CountryCodeNumber)
- **Username:** 3-20 characters, alphanumeric + underscore
- **Transaction PIN:** Exactly 4 digits (0000-9999)
- **Gender:** "male" or "female" (for profile completion)
- **Names:** Required for profile completion (first_name, last_name, middle_name optional)

---

## Environment Configuration

### **API Base URLs:**
- **Development:** `http://localhost:8000`
- **Staging:** `https://staging-api.dove.com`
- **Production:** `https://api.dove.com`

### **Feature Flags:**
- **Development Mode:** OTP returned in registration response
- **Production Mode:** OTP only sent via email

---

## Integration Checklist

### **Before Development:**
- [ ] Confirm API base URL for your environment
- [ ] Set up secure token storage
- [ ] Plan error handling strategy
- [ ] Design loading and success states

### **During Development:**
- [ ] Test all error scenarios
- [ ] Implement proper loading states
- [ ] Add input validation for all fields (email, password, passcode, username, phone, BVN, names, dates)
- [ ] Test network timeout handling
- [ ] Test phone number formatting and validation
- [ ] Implement BVN input validation (11 digits)
- [ ] Implement date picker for date of birth
- [ ] Implement transaction PIN input (4 digits, numeric keypad)
- [ ] Understand profile completion requirement for wallet creation

### **Before Release:**
- [ ] Test with production API
- [ ] Verify token security
- [ ] Test registration flow end-to-end
- [ ] Validate error message display

---

## Development vs Production

### **Development Features:**
- OTP visible in API response for testing
- Detailed error messages
- Shorter token expiration for testing

### **Production Features:**  
- OTP only sent via email
- Generic error messages for security
- Standard token expiration (60 minutes)

---

## Profile-Wallet Creation Guide

### **Wallet Creation Process:**

**Single Path (Simplified)**
```
Step 8: POST /user/update-profile → Complete Profile (All Info Stored)
Step 9: POST /kyc/create_nuban → Wallet Created (Uses Stored Profile Data)
```

### **Important Frontend Considerations:**
- **Profile completion required:** User must complete Step 8 before wallet creation
- **All fields required:** BVN, personal info, and address all needed for 9PSB
- **Clear progression:** Show users what step they're on in the registration flow
- **Validation feedback:** Provide immediate feedback on BVN format, date format, etc.
- **Secure BVN handling:** BVN is encrypted before storage for security
- **One-time wallet creation:** Once wallet is created, user has full access

---

This guide provides all the API information needed to implement the registration flow with enhanced security (password + passcode + transaction PIN) and clear virtual account creation options. The backend handles all complex business logic, security, and integrations - focus on creating a smooth user experience!