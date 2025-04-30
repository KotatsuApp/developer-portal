---
title: Architecture
description: This page provides a comprehensive overview of the Kotatsu manga reader application's architecture.
---

# Architecture

This page provides a comprehensive overview of the Kotatsu manga reader application's architecture, explaining the major components, design patterns, and interaction flows. For specific implementation details, refer to the respective feature pages such as Manga Reader, Library Management, or Manga Sources.

## High-Level Architecture

Kotatsu follows a layered architecture pattern with clear separation of concerns, implementing modern Android development practices including MVVM (Model-View-ViewModel), Repository pattern, and clean architecture principles.

## Core Components
### Build System

Kotatsu uses Gradle for build management with multiple build types (debug, release, nightly). Dependencies are managed through Gradle's version catalog system.

`TODO diagram`

### Data Layer

The data layer is responsible for data access and persistence through repositories that abstract data sources.

#### Database Structure

The application uses Room Database for persistent storage with multiple entities representing the data model.

`TODO diagram`

#### Repository System

Repositories provide a clean API for data access, abstracting the underlying data sources (local database, network, and external sources).

`TODO diagram`

### Domain Layer

The domain layer contains the business logic of the application, including use cases and business rules.

`TODO diagram`

### UI Layer

The UI layer follows the MVVM pattern with ViewModels handling UI logic and state, and Fragments/Activities for UI rendering and user interaction.

`TODO diagram`

## Data Flow

The following diagram illustrates how data flows through the application:

`TODO diagram`

## External Sources Integration

Kotatsu supports third-party manga sources through a plugin system using Android's ContentProvider mechanism:

`TODO diagram`

## Persistence and Data Backup

The application implements data backup and restore functionality through JSON serialization:

`TODO diagram`

## Database Evolution

The database schema evolves over time through migrations. The current database version is 26, with migrations handling schema changes between versions:

| Migration Version | Changes                                                                       |
| ----------------- | ----------------------------------------------------------------------------- |
| 25 to 26          | Added CloudFlare state tracking, title/cover/content rating override          |
| 24 to 25          | Added various tracking and suggestion functionality                           |
| ...               | ...                                                                           |
| 1 to 2            | Initial schema migration                                                      |

## Dependency Injection

Kotatsu uses Dagger Hilt for dependency injection, which simplifies the creation and management of dependencies throughout the application:

`TODO diagram`

## List Management and Filtering

A core aspect of the application is the management and filtering of manga lists:

`TODO diagram`

## Summary

Kotatsu's architecture follows modern Android development principles, with clear separation of concerns between UI, domain, and data layers. The application uses Room for local persistence, OkHttp for network operations, and supports external manga sources through content providers. A robust repository layer abstracts data sources, while ViewModels manage UI state using Kotlin coroutines and Flow for asynchronous operations.

This architecture promotes maintainability, testability, and extensibility, allowing for easy addition of new features and manga sources.