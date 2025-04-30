# Technology Stack

This page documents the core technologies used in the Kotatsu Sync Server and how they work together to provide synchronization capabilities. For information about the overall architecture, see [Architecture Overview](architecture-overview.md).

## Overview

The Kotatsu Sync Server is built using modern JVM technologies centered around Kotlin and its ecosystem. The server employs a layered architecture with clear separation between API routing, business logic, and data access components.

## Core Technologies
### Kotlin
Kotlin serves as the primary programming language for the server implementation. The project specifically uses Kotlin version 2.1.20 as defined in the Gradle configuration. Kotlin provides modern language features including null safety, extension functions, and coroutines that are leveraged throughout the codebase.

### Ktor Framework
Ktor (version 2.3.13) is the web framework powering the Kotatsu Sync Server. It's a lightweight, asynchronous framework designed for building connected applications in Kotlin.

Key Ktor components used in the project:

| Component                       | Purpose                                 |
| ------------------------------- | --------------------------------------- |
| ktor-server-core                | Core server functionality               |
| ktor-server-netty               | Netty engine for handling HTTP requests |
| ktor-server-auth                | Authentication support                  |
| ktor-server-auth-jwt            | JWT authentication implementation       |
| ktor-server-content-negotiation | Content negotiation for API responses   |
| ktor-serialization-kotlinx-json | JSON serialization/deserialization      |
| ktor-server-call-logging        | Request logging                         |
| ktor-server-compression         | Response compression                    |
| ktor-server-status-pages        | Error handling                          |

The server is configured to run on port 8080 by default, but this can be overridden via environment variables.

### Ktorm ORM
Ktorm (version 4.1.1) is used as the object-relational mapping (ORM) library. It provides a lightweight, flexible way to interact with databases in an object-oriented manner while still offering powerful SQL capabilities.

The project uses two Ktorm modules:

* `ktorm-core`: Provides the core ORM functionality
* `ktorm-support-mysql`: MySQL-specific extensions for Ktorm

Ktorm is used to define entity models that correspond to database tables and handle database operations throughout the application.

### Database Infrastructure
The application uses MySQL as its database system, with several supporting technologies:

* **MySQL 9.2.0**: The primary database system, used to store user data, manga information, and synchronization state
* **HikariCP 6.3.0**: A high-performance JDBC connection pool for managing database connections
* **MySQL Connector/J 9.2.0**: The JDBC driver for MySQL connectivity

The database connection is configured via environment variables or command-line arguments, with sensible defaults:

| Configuration | Default    | Environment Variable |
| ------------- | ---------- | -------------------- |
| Database Name | kotatsu_db | DATABASE_NAME        |
| Host          | localhost  | DATABASE_HOST        |
| Port          | 3306       | DATABASE_PORT        |
| User          | (Required) | DATABASE_USER        |
| Password      | (Required) | DATABASE_PASSWORD    |

## Authentication System
The authentication system is based on JSON Web Tokens (JWT) for secure, stateless authentication. The JWT implementation is configured in the application.conf file with the following parameters:

| Parameter | Description               | Configuration                           |
| --------- | ------------------------- | --------------------------------------- |
| Secret    | Signing key for tokens    | Set via JWT_SECRET environment variable |
| Issuer    | Token issuer identifier   | "http://0.0.0.0:8080/"                  |
| Audience  | Token audience identifier | "http://0.0.0.0:8080/resource"          |

The server can be configured to allow or restrict new user registrations through the `ALLOW_NEW_REGISTER` environment variable, which defaults to `true`.

## Logging
The application uses Logback Classic (version 1.5.18) for application logging. Ktor's call-logging module is implemented to log HTTP requests and responses.

## Build System
The project uses Gradle with the Kotlin DSL for build automation. Key plugins include:

| Plugin                      | Purpose                            |
| --------------------------- | ---------------------------------- |
| kotlin-jvm                  | Kotlin support for JVM             |
| ktor                        | Ktor-specific build tasks          |
| kotlin-plugin-serialization | Kotlin serialization support       |
| shadow                      | Creates fat JARs with dependencies |

The application's main class is set to `io.ktor.server.netty.EngineMain`, which is the entry point for the Ktor Netty server engine.

## Development Environment
For development purposes, the application can be run with the `-Dio.ktor.development=true` JVM argument to enable development mode features in Ktor. The Gradle build scripts are configured to set this flag automatically when running in development mode.