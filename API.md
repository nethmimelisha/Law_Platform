# Legal Services Platform API Documentation

This document provides detailed information about the API endpoints available in the Legal Services Platform backend.

## Base URL

All API endpoints are prefixed with `/api`

For local development: `http://localhost:5000/api`

## Authentication

Most endpoints require authentication using JSON Web Tokens (JWT).

Include the token in the Authorization header:
```
Authorization: Bearer YOUR_TOKEN_HERE
```

## Error Handling

All endpoints follow a standard error format:

```json
{
  "status": "error",
  "message": "Error message",
  "errors": [
    {
      "field": "email",
      "message": "Must be a valid email"
    }
  ]
}
```

## Endpoints

### Authentication

#### Register a new user

```
POST /api/auth/register
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "firstName": "John",
  "lastName": "Doe",
  "role": "client", // "client", "lawyer", or "admin"
  "phone": "+123456789",
  "address": "123 Main St",
  "city": "New York",
  "state": "NY",
  "zipCode": "10001",
  "country": "USA"
}
```

**Response (201):**
```json
{
  "status": "success",
  "token": "eyJhbGciOiJIUzI1NiIsInR5...",
  "data": {
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "role": "client",
      "phone": "+123456789",
      "isVerified": true,
      "isActive": true,
      "createdAt": "2025-03-31T12:00:00.000Z",
      "updatedAt": "2025-03-31T12:00:00.000Z"
    }
  }
}
```

#### Login

```
POST /api/auth/login
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response (200):**
```json
{
  "status": "success",
  "token": "eyJhbGciOiJIUzI1NiIsInR5...",
  "data": {
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "role": "client",
      "isVerified": true,
      "isActive": true
    }
  }
}
```

#### Get current user profile

```
GET /api/auth/me
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "role": "client",
      "phone": "+123456789",
      "address": "123 Main St",
      "city": "New York",
      "state": "NY",
      "zipCode": "10001",
      "country": "USA",
      "profilePicture": "https://example.com/image.jpg",
      "isVerified": true,
      "isActive": true,
      "createdAt": "2025-03-31T12:00:00.000Z",
      "updatedAt": "2025-03-31T12:00:00.000Z"
    }
  }
}
```

#### Update profile

```
PATCH /api/auth/update-profile
```

**Request Body:**
```json
{
  "firstName": "John",
  "lastName": "Smith",
  "phone": "+987654321",
  "address": "456 Park Ave",
  "city": "Boston",
  "state": "MA",
  "zipCode": "02108",
  "country": "USA"
}
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "firstName": "John",
      "lastName": "Smith",
      "phone": "+987654321",
      "address": "456 Park Ave",
      "city": "Boston",
      "state": "MA",
      "zipCode": "02108",
      "country": "USA",
      "email": "user@example.com",
      "role": "client"
    }
  }
}
```

#### Change password

```
PATCH /api/auth/change-password
```

**Request Body:**
```json
{
  "currentPassword": "oldPassword123",
  "newPassword": "newPassword456"
}
```

**Response (200):**
```json
{
  "status": "success",
  "message": "Password updated successfully"
}
```

### Lawyer Directory

#### Get all lawyers

```
GET /api/lawyers
```

**Query Parameters:**
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 10)
- `search`: Search by name or email
- `specialization`: Filter by specialization
- `rating`: Filter by minimum rating (1-5)
- `location`: Filter by location
- `verified`: Filter by verification status (true/false)

**Response (200):**
```json
{
  "status": "success",
  "results": 10,
  "pagination": {
    "totalItems": 50,
    "totalPages": 5,
    "currentPage": 1,
    "itemsPerPage": 10
  },
  "data": {
    "lawyers": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "firstName": "Jane",
        "lastName": "Smith",
        "email": "jane.smith@example.com",
        "phone": "+123456789",
        "city": "New York",
        "state": "NY",
        "country": "USA",
        "profilePicture": "https://example.com/image.jpg",
        "bio": "Experienced lawyer with 10+ years in corporate law",
        "averageRating": "4.8",
        "reviewsCount": "24",
        "isVerified": true,
        "specializations": [
          {
            "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
            "name": "Corporate Law"
          },
          {
            "id": "550e8400-e29b-41d4-a716-446655440001",
            "name": "Contract Law"
          }
        ]
      }
    ]
  }
}
```

#### Get lawyer details

```
GET /api/lawyers/:id
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "lawyer": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "firstName": "Jane",
      "lastName": "Smith",
      "email": "jane.smith@example.com",
      "phone": "+123456789",
      "address": "123 Legal St",
      "city": "New York",
      "state": "NY",
      "zipCode": "10001",
      "country": "USA",
      "profilePicture": "https://example.com/image.jpg",
      "bio": "Experienced lawyer with 10+ years in corporate law",
      "education": "Harvard Law School",
      "barNumber": "NY12345",
      "averageRating": "4.8",
      "reviewsCount": "24",
      "isVerified": true,
      "specializations": [
        {
          "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
          "name": "Corporate Law"
        },
        {
          "id": "550e8400-e29b-41d4-a716-446655440001",
          "name": "Contract Law"
        }
      ],
      "receivedReviews": [
        {
          "id": "f47ac10b-58cc-4372-a567-0e02b2c3d480",
          "rating": 5,
          "title": "Excellent service",
          "comment": "Very professional and knowledgeable",
          "createdAt": "2025-03-15T12:00:00.000Z",
          "client": {
            "id": "550e8400-e29b-41d4-a716-446655440002",
            "firstName": "John",
            "lastName": "Doe",
            "profilePicture": "https://example.com/avatar.jpg"
          }
        }
      ]
    }
  }
}
```

#### Get top rated lawyers

```
GET /api/lawyers/top-rated
```

**Query Parameters:**
- `limit`: Number of lawyers to return (default: 5)

**Response (200):**
```json
{
  "status": "success",
  "results": 5,
  "data": {
    "lawyers": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "firstName": "Jane",
        "lastName": "Smith",
        "averageRating": "4.9",
        "reviewsCount": "30",
        "specializations": [
          {
            "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
            "name": "Corporate Law"
          }
        ]
      }
    ]
  }
}
```

#### Get all specializations

```
GET /api/lawyers/specializations
```

**Response (200):**
```json
{
  "status": "success",
  "results": 15,
  "data": {
    "specializations": [
      {
        "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
        "name": "Corporate Law",
        "description": "Legal practice area focusing on corporation operations",
        "icon": "corporate-icon"
      }
    ]
  }
}
```

### Messaging System

#### Get conversations

```
GET /api/messages/conversations
```

**Response (200):**
```json
{
  "status": "success",
  "results": 3,
  "data": {
    "conversations": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "user": {
          "id": "550e8400-e29b-41d4-a716-446655440000",
          "firstName": "Jane",
          "lastName": "Smith",
          "profilePicture": "https://example.com/image.jpg",
          "role": "lawyer"
        },
        "lastMessage": {
          "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
          "content": "Yes, we can schedule a consultation next week",
          "createdAt": "2025-03-30T14:30:00.000Z",
          "senderId": "550e8400-e29b-41d4-a716-446655440000"
        },
        "unreadCount": 2
      }
    ]
  }
}
```

#### Get messages for a conversation

```
GET /api/messages/:userId
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "firstName": "Jane",
      "lastName": "Smith",
      "profilePicture": "https://example.com/image.jpg",
      "role": "lawyer"
    },
    "messages": [
      {
        "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
        "content": "Hello, I need legal advice regarding a contract issue",
        "senderId": "550e8400-e29b-41d4-a716-446655440002",
        "receiverId": "550e8400-e29b-41d4-a716-446655440000",
        "readAt": "2025-03-30T14:15:00.000Z",
        "createdAt": "2025-03-30T14:00:00.000Z",
        "sender": {
          "id": "550e8400-e29b-41d4-a716-446655440002",
          "firstName": "John",
          "lastName": "Doe",
          "profilePicture": "https://example.com/avatar.jpg",
          "role": "client"
        }
      },
      {
        "id": "550e8400-e29b-41d4-a716-446655440003",
        "content": "Yes, we can schedule a consultation next week",
        "senderId": "550e8400-e29b-41d4-a716-446655440000",
        "receiverId": "550e8400-e29b-41d4-a716-446655440002",
        "readAt": null,
        "createdAt": "2025-03-30T14:30:00.000Z",
        "sender": {
          "id": "550e8400-e29b-41d4-a716-446655440000",
          "firstName": "Jane",
          "lastName": "Smith",
          "profilePicture": "https://example.com/image.jpg",
          "role": "lawyer"
        }
      }
    ]
  }
}
```

### Appointments

#### Get my appointments

```
GET /api/appointments
```

**Query Parameters:**
- `startDate`: Filter by start date (ISO format)
- `endDate`: Filter by end date (ISO format)
- `status`: Filter by status (pending, confirmed, cancelled, completed)

**Response (200):**
```json
{
  "status": "success",
  "results": 2,
  "data": {
    "appointments": [
      {
        "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
        "title": "Initial Consultation",
        "description": "Discuss contract dispute details",
        "startTime": "2025-04-15T10:00:00.000Z",
        "endTime": "2025-04-15T11:00:00.000Z",
        "status": "confirmed",
        "type": "video",
        "videoUrl": "https://meet.example.com/room-id",
        "notes": "Please bring all relevant documents",
        "createdAt": "2025-03-30T12:00:00.000Z",
        "client": {
          "id": "550e8400-e29b-41d4-a716-446655440002",
          "firstName": "John",
          "lastName": "Doe",
          "email": "john.doe@example.com",
          "phone": "+123456789",
          "profilePicture": "https://example.com/avatar.jpg"
        },
        "lawyer": {
          "id": "550e8400-e29b-41d4-a716-446655440000",
          "firstName": "Jane",
          "lastName": "Smith",
          "email": "jane.smith@example.com",
          "phone": "+987654321",
          "profilePicture": "https://example.com/image.jpg"
        }
      }
    ]
  }
}
```

#### Get lawyer availability

```
GET /api/appointments/lawyer/:id/availability
```

**Query Parameters:**
- `date`: Date to check availability (YYYY-MM-DD)

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "date": "2025-04-20",
    "availableSlots": [
      {
        "startTime": "2025-04-20T09:00:00.000Z",
        "endTime": "2025-04-20T10:00:00.000Z"
      },
      {
        "startTime": "2025-04-20T11:00:00.000Z",
        "endTime": "2025-04-20T12:00:00.000Z"
      },
      {
        "startTime": "2025-04-20T14:00:00.000Z",
        "endTime": "2025-04-20T15:00:00.000Z"
      }
    ]
  }
}
```

### Reviews

#### Get lawyer reviews

```
GET /api/reviews/lawyer/:id
```

**Query Parameters:**
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 10)

**Response (200):**
```json
{
  "status": "success",
  "results": 3,
  "pagination": {
    "totalItems": 24,
    "totalPages": 3,
    "currentPage": 1,
    "itemsPerPage": 10
  },
  "data": {
    "lawyer": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "firstName": "Jane",
      "lastName": "Smith"
    },
    "stats": {
      "averageRating": "4.8",
      "totalReviews": 24
    },
    "reviews": [
      {
        "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
        "rating": 5,
        "title": "Excellent service",
        "comment": "Very professional and knowledgeable",
        "isVerified": true,
        "createdAt": "2025-03-15T12:00:00.000Z",
        "client": {
          "id": "550e8400-e29b-41d4-a716-446655440002",
          "firstName": "John",
          "lastName": "Doe",
          "profilePicture": "https://example.com/avatar.jpg"
        }
      }
    ]
  }
}
```

### Admin Dashboard

#### Get dashboard statistics

```
GET /api/users/stats/dashboard
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "users": {
      "total": 150,
      "byRole": {
        "client": 100,
        "lawyer": 45,
        "admin": 5
      }
    },
    "pendingLawyers": 12,
    "newUsersLast30Days": 35,
    "reviews": {
      "total": 120,
      "averageRating": "4.5"
    }
  }
}
```

### Case Management

#### Get user cases

```
GET /api/cases
```

**Query Parameters:**
- `status`: Filter by status
- `priority`: Filter by priority

**Response (200):**
```json
{
  "status": "success",
  "results": 3,
  "data": {
    "cases": [
      {
        "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
        "title": "Contract Dispute with XYZ Corp",
        "description": "Dispute regarding service agreement terms",
        "status": "in-progress",
        "caseType": "Contract",
        "priority": "high",
        "startDate": "2025-02-15T00:00:00.000Z",
        "client": {
          "id": "550e8400-e29b-41d4-a716-446655440002",
          "firstName": "John",
          "lastName": "Doe"
        },
        "lawyer": {
          "id": "550e8400-e29b-41d4-a716-446655440000",
          "firstName": "Jane",
          "lastName": "Smith"
        }
      }
    ]
  }
}
```
