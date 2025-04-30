---
title: Architecture Overview
description: This document provides a comprehensive overview of the Kotatsu Sync Server architecture
---

# Architecture Overview

This document provides a comprehensive overview of the Kotatsu Sync Server architecture, explaining the core components and their interactions. The architecture is designed to support synchronization of manga reading data across multiple devices using the Kotatsu App.

For specific details about technologies used, see [Technology Stack](technology-stack.md).

## System Purpose
The Kotatsu Sync Server enables users to synchronize their manga reading data (favorites, categories, and reading history) across multiple devices. The server acts as a central repository that:

1. Authenticates users
2. Stores user data securely
3. Synchronizes data between devices
4. Provides access to manga metadata

## High-Level Architecture
The architecture follows a classic client-server model with the following major components:

* **Kotatsu App**: Android client that interacts with the sync server
* **API Layer**: Handles HTTP requests and routes them to appropriate handlers
* **Authentication System**: Manages user authentication and JWT token generation
* **Data Synchronization System**: Handles the synchronization of favorites and reading history
* **User Management**: Manages user data and authentication
* **Manga Management**: Stores and retrieves manga information
* **MySQL Database**: Persistent storage for all data

## Core Components and Interactions
### Application Configuration
The Kotatsu Sync Server is configured using an application.conf file, which defines:

* Server port and deployment settings
* JWT authentication parameters
* Database connection information
* Registration settings

### Authentication System
The authentication system is built around JWT (JSON Web Tokens) for secure, stateless authentication.

Key authentication features:

* JWT tokens include user ID and have a 30-day expiration
* New user registration can be enabled/disabled via configuration
* Authenticated endpoints require a valid JWT token

### API Layer
The API layer exposes endpoints for:

| Endpoint               | Method | Purpose                                | Authentication   |
| ---------------------- | ------ | -------------------------------------- | ---------------- |
| `/auth`                | POST   | Authenticate user and obtain JWT token | :material-close: |
| `/me`                  | GET    | Get current user information           | :material-check: |
| `/resource/favourites` | GET    | Get user's favorites                   | :material-check: |
| `/resource/favourites` | POST   | Sync user's favorites                  | :material-check: |
| `/resource/history`    | GET    | Get user's reading history             | :material-check: |
| `/resource/history`    | POST   | Sync user's reading history            | :material-check: |
| `/manga/{id}`          | GET    | Get manga by ID                        | :material-close: |
| `/manga`               | GET    | Get manga list (paginated)             | :material-close: |
| `/`                    | GET    | Health check                           | :material-close: |

### Data Synchronization System
The synchronization system is at the core of the Kotatsu Sync Server. It handles synchronizing two main types of data:

1. **Favorites**: Manga titles saved as favorites, organized in categories
2. **Reading History**: Reading progress for manga titles

Key synchronization features:

* Uses database transactions with READ_COMMITTED isolation level
* Maintains timestamps for last synchronization
* Returns only changed data to minimize bandwidth usage
* Returns HTTP 204 (No Content) when no changes are needed

### Error Handling
The server implements comprehensive error handling through the StatusPages plugin:

* 400 Bad Request: For invalid input parameters
* 404 Not Found: When requested resources don't exist
* 500 Internal Server Error: For unexpected errors
All errors are logged and appropriate HTTP responses are returned to clients.

## Database Schema Interaction
The database is a central component of the architecture, storing all persistent data including:

* User accounts and authentication information
* Manga metadata (titles, URLs, ratings, cover images)
* User favorites and categories
* Reading history

The server uses the Ktorm ORM library to interact with the MySQL database, providing type-safe database operations and transaction support.

## Deployment Architecture
The Kotatsu Sync Server is designed to be deployed using either Docker or as a Systemd service.

Key deployment features:

* Environment variable configuration
* Docker Compose support for easy deployment
* Systemd support for Linux servers

## Security Considerations
Security is a critical aspect of the Kotatsu Sync Server architecture:

1. Authentication: JWT tokens with 30-day expiration
2. Password Security: Passwords are stored securely (never in plaintext)
3. Database Security: Database credentials are configurable
4. Registration Control: Optional disabling of new user registration

## Summary
The Kotatsu Sync Server architecture is designed around a modular approach with clear separation of concerns:

1. API Layer: Handles HTTP requests and responses
2. Authentication: Manages user identity and security
3. Data Synchronization: Core functionality for syncing favorites and history
4. Persistence: Database storage for all user and manga data
This architecture enables efficient synchronization of manga reading data across multiple devices while maintaining security and performance.

