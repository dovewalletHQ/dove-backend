# Dove Mobile App Registration API Guide - Frontend Developers

## Overview for Frontend Developers

This guide provides the essential API information needed to implement the new simplified Dove app registration flow in React Native. The registration flow has been completely restructured for better security and user experience.

---

## New Registration Flow Overview

### User Journey (Simplified):
```
1. Email Verification → 2. Phone Verification → 3. Username Creation → 4. Password & Account Creation → 5. Passcode Setup → 6. Profile Completion → 7. Wallet Creation
```

### Alternative: Integrated Profile + Wallet Flow:
```
1-5. Same as above → 6. Profile Completion with Wallet Creation (Single API Call)
```

### Key Features:
- **Session-based security**: All steps linked via unique session ID
- **Verification before account creation**: Email and phone verified before creating account
- **Sequential validation**: Each step validates previous steps are completed
- **Single account creation point**: Account created only after all verifications
- **Integrated wallet creation**: Profile update can create wallet in single API call
- **Flexible flow**: Support for both integrated and separate wallet creation approaches

### API Endpoints Summary:
```
POST /api/v1/auth/send-email-otp          - Send OTP to email (Step 1)
POST /api/v1/auth/verify-email-otp        - Verify email OTP (Step 2)
POST /api/v1/auth/send-phone-otp          - Send OTP to phone (Step 3)
POST /api/v1/auth/verify-phone-otp        - Verify phone OTP (Step 4)
POST /api/v1/auth/check-username          - Check username availability (Step 5)
POST /api/v1/auth/create-account          - Create account with password (Step 6)
POST /api/v1/auth/set-passcode            - Set 6-digit passcode (Step 7)
POST /api/v1/user/update-profile          - Complete profile information (Step 8)
                                           - Can create wallet with create_wallet: true
GET  /api/v1/wallet/readiness-check       - Check if ready for wallet creation (Optional)
POST /api/v1/wallet/create_wallet         - Create wallet separately (Alternative)
```

### **Flow Separation:**

**Verification Flow** (Steps 1-4):
- Email verification → Phone verification
- All data stored in session cache (30-minute expiration)
- No account creation during these steps

**Account Creation Flow** (Steps 5-6):
- Username validation → Account creation with password
- User receives authentication token after account creation

**Profile & Wallet Flow** (Steps 7-8):
- Passcode setup → Profile completion with optional wallet creation
- Requires authentication token from Step 6
- **Integrated approach**: Profile update can create wallet in single call

---

## API Endpoints Documentation

### Step 1: Send Email OTP
**Endpoint:** `POST /api/v1/auth/send-email-otp`

**Purpose:** Send 5-digit OTP verification code to user's email and initiate registration session

**Request:**
```json
{
  "email": "user@example.com"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Verification code sent to your email",
  "responseCode": "200",
  "data": {
    "email": "user@example.com",
    "session_id": "550e8400-e29b-41d4-a716-446655440000",
    "otp": "12345"  // Only in development environment
  }
}
```

**Error Responses:**
- **400:** `{"success": false, "message": "Email already registered", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to send verification code", "responseCode": "500"}`

**Notes:**
- Email is automatically converted to lowercase
- Creates unique session ID for tracking registration progress
- Session expires in 30 minutes
- OTP expires in 5 minutes
- OTP only returned in development mode for testing

---

### Step 2: Verify Email OTP
**Endpoint:** `POST /api/v1/auth/verify-email-otp`

**Purpose:** Verify email OTP using session ID and mark email as verified

**Request:**
```json
{
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "otp": "12345"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Email verified successfully",
  "responseCode": "200",
  "data": {
    "email": "user@example.com",
    "session_id": "550e8400-e29b-41d4-a716-446655440000",
    "verified": true,
    "next_step": "phone_verification"
  }
}
```

**Error Responses:**
- **400:** `{"success": false, "message": "Registration session expired or invalid", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Invalid verification code", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Invalid registration step", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to verify email", "responseCode": "500"}`

**Notes:**
- Requires session ID from Step 1
- OTP must be exactly 5 digits
- Validates that current step is "email_verification"
- Updates session to move to next step

---

### Step 3: Send Phone OTP
**Endpoint:** `POST /api/v1/auth/send-phone-otp`

**Purpose:** Send OTP to phone number using session ID (requires email to be verified)

**Request:**
```json
{
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "phone_number": "+2348123456789"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Verification code sent to your phone",
  "responseCode": "200",
  "data": {
    "session_id": "550e8400-e29b-41d4-a716-446655440000",
    "phone_number": "+2348123456789",
    "otp": "12345"  // Only in development environment
  }
}
```

**Error Responses:**
- **400:** `{"success": false, "message": "Registration session expired or invalid", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Email must be verified first", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Invalid registration step", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to send phone verification code", "responseCode": "500"}`

**Notes:**
- Phone number must be in E.164 format (+CountryCodeNumber)
- Requires email to be verified in the same session
- SMS integration will be implemented (currently returns OTP in development)
- Session ID links phone to verified email

---

### Step 4: Verify Phone OTP
**Endpoint:** `POST /api/v1/auth/verify-phone-otp`

**Purpose:** Verify phone OTP using session ID and mark phone as verified

**Request:**
```json
{
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "otp": "12345"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Phone verified successfully",
  "responseCode": "200",
  "data": {
    "session_id": "550e8400-e29b-41d4-a716-446655440000",
    "phone_number": "+2348123456789",
    "verified": true,
    "next_step": "username_selection"
  }
}
```

**Error Responses:**
- **400:** `{"success": false, "message": "Registration session expired or invalid", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Email must be verified first", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Phone number must be provided first", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Invalid verification code", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to verify phone", "responseCode": "500"}`

**Notes:**
- Validates both email and phone verification in sequence
- Updates session to allow username selection
- All verification data remains linked via session ID

---

### Step 5: Check Username Availability
**Endpoint:** `POST /api/v1/auth/check-username`

**Purpose:** Check username availability and store in session (requires email and phone verification)

**Request:**
```json
{
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "johnDoe123"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Username is available",
  "responseCode": "200",
  "data": {
    "session_id": "550e8400-e29b-41d4-a716-446655440000",
    "username": "johnDoe123",
    "available": true,
    "next_step": "account_creation"
  }
}
```

**Error Responses:**
- **400:** `{"success": false, "message": "Registration session expired or invalid", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Email must be verified first", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Phone must be verified first", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Username is already taken", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to check username availability", "responseCode": "500"}`

**Notes:**
- Username must be 3-20 characters, alphanumeric + underscore only
- Case-sensitive validation
- Requires both email and phone to be verified
- Stores username in session for account creation

---

### Step 6: Create Account
**Endpoint:** `POST /api/v1/auth/create-account`

**Purpose:** Create user account with password using all verified session data

**Request:**
```json
{
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "password": "securePassword123"
}
```

**Success Response (200):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 3600,
  "user": {
    "id": "user_id_string",
    "email": "user@example.com",
    "username": "johnDoe123",
    "phone": "+2348123456789",
    "status": "ACTIVE",
    "kyc_level": "Level_1",
    "account_tier": "Teir_1"
  },
  "message": "Account created successfully"
}
```

**Error Responses:**
- **400:** `{"success": false, "message": "Registration session expired or invalid", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Email must be verified first", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Phone must be verified first", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Username must be set first", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Email was registered by another user", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Username was taken by another user", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Account creation failed", "responseCode": "500"}`

**Notes:**
- **THIS STEP CREATES THE USER ACCOUNT** with all verified data
- Password must be minimum 8 characters
- Returns JWT access token for subsequent authenticated requests
- Performs final validation to prevent race conditions
- Cleans up registration session after successful creation
- Store the access token securely for remaining steps

---

### Step 7: Set Passcode
**Endpoint:** `POST /api/v1/auth/set-passcode`

**Purpose:** Set up a 6-digit passcode for app security (requires authentication)

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
- Required for app security and quick authentication
- User must be authenticated (access token from Step 6)

---

### Step 8: Complete Profile
**Endpoint:** `POST /api/v1/user/update-profile`

**Purpose:** Complete user profile with all information needed for wallet creation

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request (All fields optional, but all required for wallet creation):**
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
  "address_line_2": "Apartment 4B",
  "city": "Lagos",
  "state": "Lagos",
  "zip": "100001",
  "country_name": "Nigeria",
  "create_wallet": false
}
```

**Success Response (200) - Profile Updated Only:**
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

**Success Response (200) - Profile Updated with Wallet Created:**
```json
{
  "success": true,
  "message": "Profile updated successfully and wallet created",
  "responseCode": "200",
  "data": {
    "id": "user_id_string",
    "wallet": {
      "accountNumber": "2234567890",
      "fullName": "John William Doe",
      "customerID": "CUST123456",
      "bankName": "9 PAYMENT SERVICE BANK"
    }
  }
}
```

**Error Responses:**
- **401:** `{"success": false, "message": "Unauthorized", "responseCode": "401"}`
- **400:** `{"success": false, "message": "At least one field must be provided for update", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Invalid date format. Use YYYY-MM-DD", "responseCode": "400"}`
- **400:** `{"success": false, "message": "You already have a wallet created", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Please complete your profile first before creating a wallet. Missing: first_name, BVN", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to update profile", "responseCode": "500"}`
- **500:** `{"success": false, "message": "Failed to create wallet", "responseCode": "500"}`

**Notes:**
- **All fields required for wallet creation**: Complete profile needed before wallet
- **Flexible API**: Send only the fields you want to update
- **Smart validation**: Won't update fields with identical values
- **BVN encryption**: BVN is encrypted before storage for security
- **Address completion**: Full address required for banking compliance
- **Gender values**: "male" or "female"
- **Date format**: YYYY-MM-DD required
- **Phone storage**: Saved in both user.phone and kyc.phone_2
- **Country default**: Automatically set to Nigeria if not provided
- **Wallet creation integration**: Set `create_wallet: true` to create wallet after profile update
- **Wallet creation validation**: Automatically checks profile completeness before creating wallet
- **Single API call**: Can update profile and create wallet in one request when ready

---

### Step 8A: Integrated Profile Update with Wallet Creation
**Endpoint:** `POST /api/v1/user/update-profile`

**Purpose:** Complete user profile and optionally create wallet in a single API call

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request (with wallet creation):**
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
  "address_line_2": "Apartment 4B",
  "city": "Lagos",
  "state": "Lagos",
  "zip": "100001",
  "country_name": "Nigeria",
  "create_wallet": true
}
```

**Success Response (200) - Profile and Wallet Created:**
```json
{
  "success": true,
  "message": "Profile updated successfully and wallet created",
  "responseCode": "200",
  "data": {
    "id": "user_id_string",
    "wallet": {
      "accountNumber": "2234567890",
      "fullName": "John William Doe",
      "customerID": "CUST123456",
      "bankName": "9 PAYMENT SERVICE BANK"
    }
  }
}
```

**Error Response (400) - Profile Incomplete:**
```json
{
  "success": false,
  "message": "Please complete your profile first before creating a wallet. Missing: first_name, BVN",
  "responseCode": "400"
}
```

**Notes:**
- **Single API call**: Combines profile completion and wallet creation
- **Automatic validation**: Checks profile completeness before attempting wallet creation
- **Seamless experience**: User gets wallet immediately after completing profile
- **Fallback**: If `create_wallet: false` or omitted, works like regular profile update
- **Error handling**: Clear indication of missing required fields

---

### Step 9: Check Wallet Readiness
**Endpoint:** `GET /api/v1/wallet/readiness-check`

**Purpose:** Check if user has completed all required information for wallet creation

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

**Success Response (200) - Ready:**
```json
{
  "success": true,
  "message": "Wallet creation readiness checked successfully",
  "responseCode": "200",
  "data": {
    "ready_to_create": true,
    "has_wallet": false,
    "missing_fields": [],
    "total_missing": 0
  }
}
```

**Success Response (200) - Not Ready:**
```json
{
  "success": true,
  "message": "Wallet creation readiness checked successfully",
  "responseCode": "200",
  "data": {
    "ready_to_create": false,
    "has_wallet": false,
    "missing_fields": ["first_name", "last_name", "BVN", "address_line_1", "city", "state"],
    "total_missing": 6
  }
}
```

**Success Response (200) - Already Has Wallet:**
```json
{
  "success": true,
  "message": "User already has a wallet",
  "responseCode": "200",
  "data": {
    "ready_to_create": false,
    "has_wallet": true,
    "account_number": "2234567890",
    "missing_fields": []
  }
}
```

**Notes:**
- Use this endpoint to guide users on what information they need to complete
- Shows exactly which fields are missing for wallet creation
- Helps prevent failed wallet creation attempts

---

### Step 10: Create Wallet
**Endpoint:** `POST /api/v1/wallet/create_wallet`

**Purpose:** Create virtual bank account using completed profile information stored in user's account

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
  "status": "success",
  "message": "Wallet created successfully",
  "data": {
    "accountNumber": "2234567890",
    "fullName": "John William Doe",
    "customerID": "CUST123456",
    "bankName": "9 PAYMENT SERVICE BANK"
  }
}
```

**Error Responses:**
- **401:** `{"success": false, "message": "Unauthorized", "responseCode": "401"}`
- **400:** `{"success": false, "message": "You already have a wallet created", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Please complete your profile first before creating a wallet. Missing: first_name, BVN", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Failed to create wallet", "responseCode": "500"}`

**Notes:**
- **No request data needed**: All information automatically pulled from user's completed profile
- **Profile validation**: Automatically checks if all required fields are completed
- **Duplicate prevention**: Prevents creating multiple wallets for same user
- **External API**: Creates real virtual bank account via 9PSB API
- **Automatic data mapping**: 
  - Gender converted to integer (1 = male, 2 = female)
  - Phone number formatted (removes country code)
  - BVN decrypted from secure storage
  - Full address built from address components
  - Account name built from first + middle + last names
  - Transaction reference auto-generated
- **Processing time**: May take 5-10 seconds due to external banking API
- **One-time creation**: Once successful, user has full financial access

---

## Authentication & Security

### **Token Management:**
- **Token Type:** JWT Bearer token
- **Expiration:** 60 minutes (3600 seconds)
- **Storage:** Store securely in device keychain/secure storage
- **Usage:** Include in `Authorization` header for protected endpoints (Steps 7-10)

### **Session Management:**
- **Session ID:** Links all verification steps (Steps 1-6)
- **Expiration:** 30 minutes for registration session
- **Security:** Prevents race conditions and ensures data integrity
- **Cleanup:** Session automatically deleted after account creation

### **Credential Management:**
- **Password:** 8+ characters for login authentication
- **Passcode:** 6-digit security code for app access
- **Transaction PIN:** 4-digit code for financial operations (set after wallet creation)

---

## User Account States

### **Registration States:**
```
1. Email OTP Sent → SESSION_CREATED (has session_id)
2. Email Verified → EMAIL_VERIFIED (session updated)
3. Phone OTP Sent → PHONE_OTP_SENT (phone added to session)
4. Phone Verified → PHONE_VERIFIED (session updated)
5. Username Available → USERNAME_SET (session updated)
6. Account Created → ACCOUNT_ACTIVE (user created with JWT token)
7. Passcode Set → PASSCODE_SET (security enabled)
8. Profile Completed → PROFILE_COMPLETE (ready for wallet)
9. Wallet Created → FULL_ACCESS (complete financial access)
```

### **Key Differences from Old Flow:**
- **Session-based**: All data linked via session ID until account creation
- **Verification first**: Email and phone verified before account exists
- **Single creation point**: Account created only in Step 6 with all verified data
- **Better security**: Prevents partial registrations and race conditions

---

## Error Handling Guide

### **Session Errors:**
- **Session Expired:** Restart registration from Step 1
- **Invalid Session:** Show error and restart registration
- **Wrong Step:** Guide user to correct step in sequence

### **Validation Errors:**
- **Invalid Email:** Show proper email format message
- **Weak Password:** Display password requirements (8+ characters)
- **Invalid Passcode:** Show passcode format error (must be 6 digits)
- **Invalid OTP:** Clear OTP input and show error
- **Expired OTP:** Show resend option
- **Username Taken:** Suggest alternatives

### **Common Error Response Format:**
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
- **Email/Phone OTP:** 1-3 seconds
- **OTP Verification:** 1-2 seconds
- **Username Check:** 1-2 seconds
- **Account Creation:** 2-3 seconds
- **Profile Update:** 1-2 seconds
- **Wallet Creation:** 5-10 seconds (external banking API)

### **Timeout Recommendations:**
- Set request timeout to 15 seconds for wallet creation
- Set request timeout to 10 seconds for other endpoints
- Show loading states for requests over 2 seconds

---

## Mobile App Implementation Guide

### **Step-by-Step Implementation:**

1. **Email Verification Screen:**
   - Email input → Send OTP → Store session_id
   - OTP input → Verify → Store verified status

2. **Phone Verification Screen:**
   - Phone input → Send OTP (with session_id)
   - OTP input → Verify → Update session

3. **Username Screen:**
   - Username input → Check availability (with session_id)
   - Show availability status

4. **Password Screen:**
   - Password input → Create account (with session_id)
   - Store JWT token securely

5. **Passcode Screen:**
   - 6-digit passcode input → Set passcode (with JWT)

6. **Profile Screen (Integrated Approach):**
   - Multi-step form → Update profile with `create_wallet: true` (with JWT)
   - Show wallet details immediately after successful profile completion
   - Handle wallet creation errors gracefully

7. **Alternative: Separate Wallet Screen:**
   - Check readiness → Create wallet button (empty request body)
   - Show wallet details after successful creation

### **Data Flow Management:**
```javascript
// Example state management
const registrationState = {
  sessionId: null,          // From Step 1
  email: null,              // From Step 1
  emailVerified: false,     // From Step 2
  phone: null,              // From Step 3
  phoneVerified: false,     // From Step 4
  username: null,           // From Step 5
  accountCreated: false,    // From Step 6
  accessToken: null,        // From Step 6
  passcodeSet: false,       // From Step 7
  profileComplete: false,   // From Step 8
  walletCreated: false,     // From Step 8A (integrated) or Step 10 (separate)
  walletData: null          // Store wallet information
};
```

### **Validation Requirements:**
- **Email:** Valid email format
- **Phone:** E.164 format (+CountryCodeNumber)
- **OTP:** Exactly 5 digits (numeric only)
- **Username:** 3-20 characters, alphanumeric + underscore
- **Password:** Minimum 8 characters
- **Passcode:** Exactly 6 digits (numeric only)
- **BVN:** Exactly 11 digits (numeric only)
- **Date of Birth:** YYYY-MM-DD format
- **Gender:** "male" or "female"
- **Names:** Required for wallet creation (first_name, last_name)

---

## Environment Configuration

### **API Base URLs:**
- **Development:** `http://localhost:8000`
- **Staging:** `https://staging-api.dove.com`
- **Production:** `https://api.dove.com`

### **Feature Flags:**
- **Development Mode:** OTP returned in API responses for testing
- **Production Mode:** OTP only sent via email/SMS

---

## Integration Checklist

### **Before Development:**
- [ ] Confirm API base URL for your environment
- [ ] Set up secure token storage (Keychain/Keystore)
- [ ] Plan session state management
- [ ] Design error handling strategy
- [ ] Plan loading and success states

### **During Development:**
- [ ] Implement session ID tracking across steps
- [ ] Test all error scenarios and session expiration
- [ ] Implement proper loading states for each step
- [ ] Add input validation for all fields
- [ ] Test network timeout handling
- [ ] Implement step-by-step navigation
- [ ] Add progress indicators
- [ ] Test wallet readiness checking

### **Before Release:**
- [ ] Test complete registration flow end-to-end
- [ ] Test with production API
- [ ] Verify secure token storage
- [ ] Test session expiration scenarios
- [ ] Validate error message display
- [ ] Test wallet creation with complete profile

---

This new registration flow provides better security, clearer user experience, and more reliable account creation. The session-based approach ensures all verification happens before account creation, preventing incomplete registrations and improving data integrity.