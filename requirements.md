# 📘 Backend Requirement Specifications

This document outlines the **functional and technical specifications** for the backend of the Airbnb-like rental platform, covering three core features:

- **User Authentication**
- **Property Management**
- **Booking System**

---

## 1. 🔐 User Authentication

### Feature Description

Allow users (guests or hosts) to register, log in, and manage secure sessions using JWT tokens. Role-based access control ensures different access levels for guests, hosts, and admins.

### API Endpoints

| Method | Endpoint             | Description             |
| ------ | -------------------- | ----------------------- |
| POST   | `/api/auth/register` | Register a new user     |
| POST   | `/api/auth/login`    | Authenticate user login |

### Input/Output Specifications

#### `POST /api/auth/register`

- **Input**:

```json
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "password": "securePass123",
  "role": "guest"
}
```

- **Validations**:
  - Unique email (validated server-side)
  - Password ≥ 8 characters
  - Role: must be one of `guest`, `host`, `admin`
- **Output**:

```json
{
  "token": "<JWT>",
  "user": { "id": "...", "email": "...", "role": "guest" }
}
```

#### `POST /api/auth/login`

- **Input**:

```json
{
  "email": "john@example.com",
  "password": "securePass123"
}
```

- **Output**:

```json
{
  "token": "<JWT>",
  "user": { "id": "...", "email": "...", "role": "guest" }
}
```

### Performance Criteria

- Response time: ≤ 200ms
- Rate limit login attempts (e.g., 5 per minute/IP)

---

## 2. 🏡 Property Management

### Feature Description

Hosts can create, edit, and delete property listings with full details including location, price, and description.

### API Endpoints

| Method | Endpoint              | Description               |
| ------ | --------------------- | ------------------------- |
| POST   | `/api/properties`     | Add new property          |
| GET    | `/api/properties`     | List/search properties    |
| GET    | `/api/properties/:id` | Retrieve property details |
| PUT    | `/api/properties/:id` | Update property           |
| DELETE | `/api/properties/:id` | Delete property           |

### Input/Output Specifications

#### `POST /api/properties`

- **Input**:

```json
{
  "name": "Cozy Apartment",
  "description": "Nice 2-bedroom in downtown",
  "location": "Cairo, Egypt",
  "pricepernight": 350
}
```

- **Output**:

```json
{
  "property_id": "...",
  "message": "Property created successfully"
}
```

### Validation Rules

- Name and description required
- Price ≥ 0
- Host must be authenticated (JWT)

### Performance Criteria

- Full list paginated (default 10/page)
- Searchable by location and price

---

## 3. 📆 Booking System

### Feature Description

Guests can request to book available properties for specific dates. The system prevents double bookings and manages booking status.

### API Endpoints

| Method | Endpoint                   | Description       |
| ------ | -------------------------- | ----------------- |
| POST   | `/api/bookings`            | Create a booking  |
| GET    | `/api/bookings/user`       | Get user bookings |
| PUT    | `/api/bookings/:id/cancel` | Cancel a booking  |

### Input/Output Specifications

#### `POST /api/bookings`

- **Input**:

```json
{
  "property_id": "...",
  "start_date": "2025-07-05",
  "end_date": "2025-07-10"
}
```

- **Output**:

```json
{
  "booking_id": "...",
  "status": "pending",
  "total_price": 1750
}
```

### Validation Rules

- Dates must be in the future
- No overlapping bookings allowed
- User must be authenticated (JWT)

### Booking Status Flow

- `pending` → `confirmed` (after payment)
- `confirmed` → `canceled` (by guest/host)

### Performance Criteria

- Conflict checks on date ranges within 150ms
- List retrieval paginated, sorted by start date

---

## 🛠 Technical Assumptions

- **Database**: PostgreSQL or MySQL with normalized schema
- **Auth**: JWT with Role-based access control
- **Storage**: Images stored using file system or AWS S3
- **Errors**: Standardized JSON error response + logging

