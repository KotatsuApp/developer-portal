---
title: API Documentation
description: This page documents the API endpoints provided by the Kotatsu Sync Server, detailing the available routes, etc.
---

# API Documentation

This page documents the API endpoints provided by the Kotatsu Sync Server, detailing the available routes, request/response formats, and authentication mechanisms. For information about server deployment, see Installation and Deployment, and for detailed synchronization implementation details, see Core Features.

## API Overview
The Kotatsu Sync Server provides REST API endpoints for user authentication and data synchronization between multiple devices. All synchronization endpoints require authentication using JWT tokens.

## Authentication
The API uses JWT (JSON Web Token) for authentication. Clients must first obtain a token by sending credentials to the /auth endpoint, then include this token in the Authorization header for subsequent requests.

### Auth Endpoint

```
POST /auth
Content-Type: application/json
```

Request body:

```
{
  "email": "user@example.com",
  "password": "password123",
  "nickname": "username"  // Optional for new registrations
}
```

Response:

```
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

Notes:

* The server can be configured to allow or disallow new user registrations
* JWT tokens are valid for 30 days
* If authentication fails, the server responds with 400 Bad Request

## Synchronization Endpoints
### Favorites Synchronization

```
POST /resource/favourites
Authorization: Bearer {token}
Content-Type: application/json
```

Request body: `FavouritesPackage` containing categories and favorites data.

Response:

* 204 No Content if server accepts client data without changes
* 200 OK with updated FavouritesPackage if server has newer data

```
GET /resource/favourites
Authorization: Bearer {token}
```

Response:

* 200 OK with FavouritesPackage containing all user's favorites

### History Synchronization

```
POST /resource/history
Authorization: Bearer {token}
Content-Type: application/json
```

Request body: HistoryPackage containing reading history data.

Response:

* 204 No Content if server accepts client data without changes
* 200 OK with updated HistoryPackage if server has newer data

```
GET /resource/history
Authorization: Bearer {token}
```

Response:

* 200 OK with HistoryPackage containing user's reading history

## User Information

```
GET /me
Authorization: Bearer {token}
```

Response:

* 200 OK with user information
* 401 Unauthorized if token is invalid

## Manga Endpoints
### Get Manga by ID

```
GET /manga/{id}
```

Parameters:

* `id` - Manga ID (numeric)

Response:

* 200 OK with manga data
* 404 Not Found if manga doesn't exist

### Get Manga List

```
GET /manga?offset={offset}&limit={limit}
```

Parameters:

* `offset` - Number of items to skip (required)
* `limit` - Maximum number of items to return (required)

Response:

* 200 OK with array of manga data
* 400 Bad Request if parameters are missing or invalid

## Error Handling
The API uses standard HTTP status codes to indicate the result of requests:

| Status Code               | Description                          | Common Causes                        |
| ------------------------- | ------------------------------------ | ------------------------------------ |
| 200 OK                    | Request successful                   | -                                    |
| 204 No Content            | Request successful, no response body | Server accepted data without changes |
| 400 Bad Request           | Invalid request                      | Missing parameters, invalid format   |
| 401 Unauthorized          | Authentication required              | Missing or invalid JWT token         |
| 404 Not Found             | Resource not found                   | Invalid manga ID                     |
| 500 Internal Server Error | Server error                         | Unexpected server issues             |

## Authentication Implementation
The server uses the Ktor authentication framework with JWT for secure authentication. The JWT token contains:

* Audience claim
* Issuer claim
* User ID claim
* Expiration date (30 days from creation)

The token must be included in the `Authorization` header as a Bearer token:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

JWT configuration values are defined in the application configuration:

* `jwt.secret` - Secret key for signing tokens
* `jwt.issuer` - Token issuer
* `jwt.audience` - Token audience
* `kotatsu.allow_new_register` - Whether to allow new user registration

## Synchronization Mechanism
The synchronization endpoints follow a consistent pattern:

1. Client sends local data with timestamps
2. Server compares with stored data
3. Server decides which data is newer (client's or server's)
4. If server accepts client data, it updates its database and returns 204 No Content
5. If server has newer data, it returns 200 OK with the updated data
6. Server updates synchronization timestamps after successful synchronization
