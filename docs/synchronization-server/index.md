# Overview

This document provides a comprehensive introduction to the Kotatsu Sync Server, an open-source synchronization server for the 
Kotatsu app. The server enables users to synchronize their favorites, reading history, and categories across multiple devices, providing a seamless manga reading experience.

For detailed installation instructions, see Installation and Deployment, and for API documentation, see API Documentation.

!!! danger

    This function is experimental. Please make sure you have a backup to avoid data loss.

## What is Kotatsu Sync Server?

Kotatsu Sync Server is a backend service that allows Kotatsu app users to store their reading data in a central location and synchronize it across multiple devices. The server provides a secure API for data synchronization, user authentication, and data management.

## Synchronizable Data
The Kotatsu Sync Server supports synchronization of the following data types:

1. **Favorites** - Manga saved to a user's favorites collection
2. **Categories** - Custom groupings for organizing favorite manga
3. **History** - Reading progress and history for each manga

## System Architecture
The Kotatsu Sync Server follows a layered architecture pattern, with clear separation between API, business logic, and data access layers.

## Core Components
### Application Configuration
The server is initialized in the `Application.kt` file, where various plugins and modules are configured:

* Authentication with JWT
* Database connection
* Serialization for JSON handling
* Error handling via status pages
* API routing

### Authentication System
The authentication system uses JWT (JSON Web Tokens) to secure API endpoints. Users provide their credentials (email and password) to obtain a token that is then used for subsequent requests.

### Data Synchronization System
The data synchronization system is responsible for keeping user data consistent across multiple devices. It handles three main types of data:

1. **Favorites Synchronization** - Manages the user's favorite manga and categories
2. **History Synchronization** - Tracks reading progress across devices
3. **Manga Data Synchronization** - Ensures manga metadata is consistent

## Error Handling
The server implements comprehensive error handling through status pages. Different types of exceptions are mapped to appropriate HTTP status codes:

* `IllegalArgumentException` - 400 Bad Request
* `NotFoundException` - 404 Not Found
* Other exceptions - 500 Internal Server Error
This ensures consistent error responses across the API.

## Database Schema
The sync server relies on a MySQL database with the following main tables:

| Table      | Description                                                                    |
| ---------- | ------------------------------------------------------------------------------ |
| USERS      | Stores user account information including email, password, and sync timestamps |
| MANGA      | Contains metadata about manga titles                                           |
| CATEGORIES | Stores user-defined categories for organizing favorites                        |
| FAVOURITES | Maps manga to users and categories, with timestamps for sync                   |
| HISTORY    | Records reading progress for each manga per user                               |
| TAGS       | Stores manga tags/genres                                                       |
| MANGA_TAGS | Maps tags to manga                                                             |

## Deployment Options
The server can be deployed in several ways:

1. **Docker** - Containerized deployment with configuration via environment variables
2. **Systemd** - Traditional deployment as a system service on Linux
For detailed deployment instructions, see Docker Deployment and Systemd Deployment.

