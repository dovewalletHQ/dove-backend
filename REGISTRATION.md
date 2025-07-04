# 📱 **Dove Mobile App Registration API Guide - Frontend Developers**

## **🎯 Overview for Frontend Developers**

This guide provides the essential API information needed to implement the Dove app registration flow in React Native. No code examples - just the API specifications, data flow, and integration requirements.

---

## **🔥 Registration Flow Overview**

### **📋 User Journey:**
```
1. Email Registration → 2. OTP Verification & Account Creation (Password + Passcode) → 3. Profile Setup → 4. Wallet Creation
```

### **📊 API Endpoints Summary:**
```
POST /api/v1/auth/register          - Send OTP to email
POST /api/v1/auth/verify-otp        - Verify OTP & create account  
PUT  /api/v1/kyc/address           - Update user profile (optional)
POST /api/v1/kyc/create_nuban      - Create virtual bank account
```

---

## **📡 API Endpoints Documentation**

### **Step 1: Email Registration**
**Endpoint:** `POST /api/v1/auth/register`

**Purpose:** Send OTP verification code to user's email

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
  "message": "OTP sent successfully",
  "responseCode": "200",
  "data": {
    "email": "user@example.com",
    "otp": "123456"  // Only in development environment
  }
}
```

**Error Responses:**
- **400:** `{"success": false, "message": "User already exists", "responseCode": "400"}`
- **500:** `{"success": false, "message": "Registration failed", "responseCode": "500"}`

**Notes:**
- Email is automatically converted to lowercase
- OTP expires after 5 minutes (configurable)
- OTP is only returned in development mode for testing

---

### **Step 2: OTP Verification & Account Creation**
**Endpoint:** `POST /api/v1/auth/verify-otp`

**Purpose:** Verify OTP and create complete user account with security credentials

**Request:**
```json
{
  "email": "user@example.com",
  "otp": "123456",
  "password": "securePassword123",
  "username": "johnDoe",
  "passcode": "123456"
}
```

**Success Response (200):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 3600,
  "user": "{\"id\":\"...\",\"email\":\"user@example.com\",\"username\":\"johnDoe\",...}"
}
```

**Error Responses:**
- **400:** `{"success": false, "message": "Invalid or expired OTP", "responseCode": "400"}`
- **400:** `{"success": false, "message": "Email already registered", "responseCode": "400"}`
- **500:** `{"success": false, "message": "User creation failed", "responseCode": "500"}`

**Notes:**
- This endpoint creates the user account AND returns authentication token
- Store the `access_token` securely for subsequent API calls
- Both password and passcode are hashed automatically by the backend
- User status is set to "ACTIVE" after successful verification
- Passcode is a 6-digit security code separate from the login password

---

### **Step 3: Profile Setup (Optional)**
**Endpoint:** `PUT /api/v1/kyc/address`

**Purpose:** Update user profile with address information

**⚠️ IMPORTANT:** This endpoint automatically creates a virtual bank account

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request:**
```json
{
  "address_line_1": "123 Main Street",
  "address_line_2": "Apt 4B",
  "city": "Lagos",
  "state": "Lagos",
  "zip": "100001",
  "country_name": "Nigeria"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "Profile updated successfully",
  "responseCode": "200"
}
```

**Error Responses:**
- **401:** `{"success": false, "message": "Unauthorized", "responseCode": "401"}`
- **400:** `{"success": false, "message": "Invalid address data", "responseCode": "400"}`

**Notes:**
- This step is optional but recommended
- All address fields are optional
- User cache is automatically updated after successful update
- **AUTOMATIC NUBAN CREATION:** Completing this step immediately creates a virtual bank account
- User is upgraded to KYC Level_2 and Account Tier_2

---

### **Step 4: Virtual Account Creation**
**Endpoint:** `POST /api/v1/kyc/create_nuban`

**Purpose:** Create virtual bank account for the user (Alternative to Step 3)

**Headers Required:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request:**
```json
{
  "address": {
    "address_line_1": "123 Main Street",
    "city": "Lagos",
    "state": "Lagos",
    "country_name": "Nigeria"
  }
}
```

**Success Response (200):**
```json
{
  "success": true,
  "message": "NUBAN account created successfully",
  "responseCode": "200",
  "data": {
    "account_number": "2234567890",
    "account_name": "John Doe",
    "bank_name": "9PSB",
    "customer_id": "CUST123456"
  }
}
```

**Error Responses:**
- **401:** `{"success": false, "message": "Unauthorized", "responseCode": "401"}`
- **400:** `{"success": false, "message": "Address information required", "responseCode": "400"}`
- **502:** `{"success": false, "message": "Banking service unavailable", "responseCode": "502"}`

**Notes:**
- This creates a real virtual bank account via 9PSB
- Account number can be used to receive payments
- Address information is required for account creation
- Process may take a few seconds due to external API call
- **USE CASE:** Use this endpoint if you skipped Step 3 but want to create a virtual account
- **EITHER/OR:** Users can either do Step 3 (automatic) OR Step 4 (manual), not both

---

## **🔐 Authentication & Security**

### **Token Management:**
- **Token Type:** JWT Bearer token
- **Expiration:** 60 minutes (3600 seconds)
- **Storage:** Store securely in device keychain/secure storage
- **Usage:** Include in `Authorization` header for protected endpoints

### **Credential Management:**
- **Password:** 8+ characters for login authentication
- **Passcode:** 6-digit security code (set during registration)
- **Transaction PIN:** 4-digit code for financial operations (set later by user)

### **Token Format:**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### **Session Management:**
- User session is cached on backend for performance
- Token automatically refreshed on successful API calls
- Logout by deleting token from device storage

---

## **📊 User Account States**

### **Registration States:**
```
1. Email Submitted → PENDING (OTP sent)
2. OTP Verified → ACTIVE (account created with password + passcode)
3. Profile Updated → ACTIVE (optional - auto-creates wallet)
4. Manual Wallet Created → ACTIVE (alternative financial setup)
```

### **Account Levels:**
- **KYC Level:** LEVEL_1 (default) → LEVEL_2 → LEVEL_3
- **Account Tier:** TIER_1 (default) → TIER_2 → TIER_3
- **Status:** PENDING → ACTIVE

---

## **🚨 Error Handling Guide**

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

## **⏱️ Timing & Performance**

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

## **🔄 Additional Authentication Endpoints**

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

## **📱 Mobile App Considerations**

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
- **Passcode:** Exactly 6 digits (numeric only)
- **Username:** 3-20 characters, alphanumeric + underscore
- **OTP:** Exactly 6 digits

---

## **🌍 Environment Configuration**

### **API Base URLs:**
- **Development:** `http://localhost:8000`
- **Staging:** `https://staging-api.dove.com`
- **Production:** `https://api.dove.com`

### **Feature Flags:**
- **Development Mode:** OTP returned in registration response
- **Production Mode:** OTP only sent via email

---

## **📋 Integration Checklist**

### **Before Development:**
- [ ] Confirm API base URL for your environment
- [ ] Set up secure token storage
- [ ] Plan error handling strategy
- [ ] Design loading and success states

### **During Development:**
- [ ] Test all error scenarios
- [ ] Implement proper loading states
- [ ] Add input validation for all fields (email, password, passcode, username)
- [ ] Test network timeout handling
- [ ] Understand address/NUBAN coupling for user flow design

### **Before Release:**
- [ ] Test with production API
- [ ] Verify token security
- [ ] Test registration flow end-to-end
- [ ] Validate error message display

---

## **🛠️ Development vs Production**

### **Development Features:**
- OTP visible in API response for testing
- Detailed error messages
- Shorter token expiration for testing

### **Production Features:**  
- OTP only sent via email
- Generic error messages for security
- Standard token expiration (60 minutes)

---

## **🔗 Address-NUBAN Coupling Guide**

### **Two Paths for Virtual Account Creation:**

**Path A: Automatic (Recommended)**
```
Step 3: PUT /kyc/address → Address Updated + NUBAN Created Automatically
```

**Path B: Manual**
```
Skip Step 3 → Step 4: POST /kyc/create_nuban → NUBAN Created with Address in Payload
```

### **Important Frontend Considerations:**
- **Cannot separate address and NUBAN:** If user updates address, NUBAN is created immediately
- **Show clear messaging:** Inform users that providing address will create their bank account
- **Either/or choice:** Design UI to show user can either do automatic (address) or manual (direct NUBAN)
- **No reversal:** Once NUBAN is created via address update, it cannot be undone

---

This guide provides all the API information needed to implement the registration flow with enhanced security (password + passcode) and clear virtual account creation options. The backend handles all complex business logic, security, and integrations - focus on creating a smooth user experience! 🚀