
# 🔐 Backend Feature Specification – Airbnb Clone

## 1. 🧍 User Authentication

### 📘 Feature Description
Handles secure registration, login, and session management for users (guests and hosts) using JWT. Also supports third-party OAuth logins.

---

### 📡 API Endpoints

#### POST `/api/auth/register`
Registers a new user (guest or host).

#### POST `/api/auth/login`
Authenticates user and returns JWT token.

#### GET `/api/auth/profile`
Returns the authenticated user's profile (requires JWT).

#### POST `/api/auth/oauth/google`
Handles OAuth login via Google (optional extension).

---

### 📥 Input Specifications

#### `/register` JSON Body
```json
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "password": "SecurePass123!",
  "role": "guest"
}
```

#### `/login` JSON Body
```json
{
  "email": "john@example.com",
  "password": "SecurePass123!"
}
```

---

### 📤 Output Specifications

#### Successful Login
```json
{
  "token": "jwt_token_string",
  "user": {
    "id": "uuid",
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com",
    "role": "guest"
  }
}
```

#### Error Response
```json
{
  "error": "Invalid credentials"
}
```

---

### ✅ Validation Rules
- Email must be valid and unique
- Password must be at least 8 characters, include numbers and symbols
- Role must be either `guest` or `host`
- Duplicate emails must return `409 Conflict`

---

### 🚀 Performance Criteria
- Token generation within 150ms
- Login endpoint response ≤ 200ms under normal load
- Rate-limiting on login: max 5 attempts per minute/IP
- Token expiration: 24 hours (refresh tokens optional)

---

## 2. 🏘️ Property Management

### 📘 Feature Description
Allows hosts to create, update, and delete property listings with details like price, description, images, and availability.

---

### 📡 API Endpoints

#### POST `/api/properties`
Create a new listing (host only).

#### GET `/api/properties`
Retrieve all listings (with optional filters).

#### GET `/api/properties/:id`
Retrieve a specific property.

#### PUT `/api/properties/:id`
Update listing details (host only).

#### DELETE `/api/properties/:id`
Delete a listing (host only).

---

### 📥 Input Specifications

#### `/api/properties` JSON Body
```json
{
  "title": "Cozy Apartment in Accra",
  "description": "A modern 2-bedroom apartment...",
  "price_per_night": 80,
  "location": "Accra",
  "amenities": ["WiFi", "AC", "TV"],
  "max_guests": 4,
  "availability": {
    "start_date": "2025-07-01",
    "end_date": "2025-08-31"
  }
}
```

---

### 📤 Output Specifications

#### Successful Creation
```json
{
  "message": "Property listed successfully",
  "property_id": "uuid"
}
```

---

### ✅ Validation Rules
- Title: min 5, max 100 characters
- Price must be > 0
- Max guests: must be a positive integer
- Availability dates must be valid ISO strings
- User must be authenticated as a `host`

---

### 🚀 Performance Criteria
- List all properties (w/ filters) in < 500ms
- Cache popular search queries using Redis
- Image uploads handled via async job (stored in local or S3)
- Support pagination: limit, offset

---

## 3. 📅 Booking System

### 📘 Feature Description
Allows guests to book available properties, ensures no double-booking, tracks booking status, and supports cancellations.

---

### 📡 API Endpoints

#### POST `/api/bookings`
Create a new booking.

#### GET `/api/bookings`
Get user's bookings (guest or host view).

#### PUT `/api/bookings/:id/cancel`
Cancel a booking (if allowed).

#### GET `/api/bookings/:id`
Get detailed booking info.

---

### 📥 Input Specifications

#### `/api/bookings` JSON Body
```json
{
  "property_id": "uuid",
  "start_date": "2025-07-15",
  "end_date": "2025-07-20"
}
```

---

### 📤 Output Specifications

#### Successful Booking
```json
{
  "message": "Booking confirmed",
  "booking_id": "uuid",
  "status": "pending"
}
```

#### Conflict Response
```json
{
  "error": "Property not available for selected dates"
}
```

---

### ✅ Validation Rules
- Start date must be before end date
- Property must be available for entire date range
- Guest must be authenticated
- Cannot book own property
- Cancellation only allowed up to 48 hours before check-in

---

### 🚀 Performance Criteria
- Booking response ≤ 300ms
- Conflict checks use indexed date ranges in DB
- Prevent race conditions with row-level locking or transactions
- Booking status automatically changes to `confirmed` after payment

---

# 📝 Notes
- All endpoints should return proper HTTP status codes (e.g., 200, 201, 400, 401, 409, 500)
- API secured via Bearer Token in Authorization header
- JSON is the standard format for all request/response payloads
- Pagination, filtering, and sorting should be supported where applicable
