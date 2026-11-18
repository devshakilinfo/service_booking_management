# 📡 ServiceHub REST API Documentation (PostgreSQL)

## 1. Overview & Base URLs
The ServiceHub Backend provides a RESTful JSON API secured with JSON Web Tokens (JWT).

- **Local Development Base URL:** `http://localhost:5000/api`
- **Android Emulator Base URL:** `http://10.0.2.2:5000/api`
- **Network / Real Device Base URL:** `http://<YOUR_LOCAL_IP>:5000/api`

### Standard Response Format
```json
{
  "success": true,
  "message": "Optional message",
  "data": { ... }
}
```

---

## 2. Authentication Header
Protected endpoints require:
```http
Authorization: Bearer <your_jwt_token>
```

---

## 3. Endpoints

### 🔐 A. Authentication API

#### 1. Register New User
* **Route:** `POST /api/auth/register`
* **Access:** Public
* **Request Body:**
```json
{
  "name": "Shakil Ahamed",
  "email": "shakil@example.com",
  "password": "password123"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Registration successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "name": "Shakil Ahamed",
    "email": "shakil@example.com",
    "phone": "",
    "avatar": "https://images.unsplash.com/photo-1494790108377-be9c29b29330?w=150",
    "role": "user"
  }
}
```

#### 2. User Login
* **Route:** `POST /api/auth/login`
* **Access:** Public
* **Request Body:**
```json
{
  "email": "shakil@example.com",
  "password": "password123"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "name": "Shakil Ahamed",
    "email": "shakil@example.com",
    "phone": "+1 234 567 8900",
    "avatar": "https://images.unsplash.com/photo-1494790108377-be9c29b29330?w=150",
    "role": "user"
  }
}
```

---

### 🛠️ B. Service Management API

#### 1. Get All Services (with Filtering & Search)
* **Route:** `GET /api/services`
* **Access:** Public
* **Query Parameters:**
  * `category` *(optional)*: Filter by category (e.g. `Cleaning`, `Repair`, `Beauty`, `Electric`)
  * `search` *(optional)*: Full-text search keyword in service name, description, or category
  * `featured` *(optional)*: `true` / `false`
  * `popular` *(optional)*: `true` / `false`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "count": 1,
  "data": [
    {
      "id": "1",
      "name": "Home Cleaning",
      "description": "Professional home cleaning service for a fresh and healthy home.",
      "category": "Cleaning",
      "price": 25.00,
      "duration": "1 hr",
      "image": "https://images.unsplash.com/photo-1581578731548-c64695cc6952?w=800",
      "rating": 4.8,
      "reviewsCount": 2500,
      "isFeatured": true,
      "isPopular": true,
      "highlights": ["Experienced Staff", "Eco-Friendly Products", "100% Satisfaction"],
      "availableSlots": ["09:00 AM", "10:00 AM", "11:00 AM", "02:00 PM", "03:00 PM", "04:00 PM"]
    }
  ]
}
```

#### 2. Get Single Service by ID
* **Route:** `GET /api/services/:id`
* **Access:** Public
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "1",
    "name": "Home Cleaning",
    "description": "Professional home cleaning service...",
    "category": "Cleaning",
    "price": 25.00,
    "duration": "1 hr",
    "rating": 4.8,
    "reviewsCount": 2500
  }
}
```

#### 3. Check Slot Availability for a Date
* **Route:** `GET /api/services/:id/availability?date=YYYY-MM-DD`
* **Access:** Public
* **Success Response (200 OK):**
```json
{
  "success": true,
  "date": "2026-11-20",
  "totalSlots": 6,
  "availableSlotsCount": 5,
  "slots": [
    { "time": "09:00 AM", "isAvailable": true },
    { "time": "10:00 AM", "isAvailable": false },
    { "time": "11:00 AM", "isAvailable": true },
    { "time": "02:00 PM", "isAvailable": true },
    { "time": "03:00 PM", "isAvailable": true },
    { "time": "04:00 PM", "isAvailable": true }
  ]
}
```

---

### 📅 C. Appointment Management API

#### 1. Book an Appointment
* **Route:** `POST /api/bookings`
* **Access:** Private (Bearer Token)
* **Request Body:**
```json
{
  "serviceId": 1,
  "date": "2026-11-20",
  "timeSlot": "09:00 AM",
  "notes": "Please bring eco-friendly cleaning detergents"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Appointment booked successfully",
  "data": {
    "id": "1",
    "date": "2026-11-20",
    "timeSlot": "09:00 AM",
    "status": "Pending",
    "totalPrice": 25.00,
    "service": {
      "id": "1",
      "name": "Home Cleaning",
      "category": "Cleaning",
      "price": 25.00,
      "duration": "1 hr"
    }
  }
}
```
* **Conflict Error (409 Conflict):**
```json
{
  "success": false,
  "message": "This time slot is already booked. Please choose another time slot."
}
```

#### 2. Get User Bookings
* **Route:** `GET /api/bookings/my`
* **Access:** Private (Bearer Token)
* **Query Parameters:** `type=upcoming` or `type=past`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "count": 1,
  "data": [
    {
      "id": "1",
      "date": "2026-11-20",
      "timeSlot": "09:00 AM",
      "status": "Confirmed",
      "totalPrice": 25.00,
      "service": {
        "id": "1",
        "name": "Home Cleaning",
        "category": "Cleaning",
        "price": 25.00,
        "duration": "1 hr"
      }
    }
  ]
}
```

#### 3. Cancel Appointment
* **Route:** `PUT /api/bookings/:id/cancel`
* **Access:** Private (Bearer Token)
* **Request Body:**
```json
{
  "cancellationReason": "Need to reschedule"
}
```

---

### 👤 D. User Profile API

* `GET /api/users/profile` - Get current user profile
* `PUT /api/users/profile` - Update profile and change password

---

### 🤖 E. AI Service Assistant API

* `POST /api/ai/ask` - Ask question about services & get grounded recommendations with service cards.
