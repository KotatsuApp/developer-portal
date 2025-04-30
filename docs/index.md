# Overview

Kotatsu is an open-source manga reader application for Android that allows users to read manga from various online sources and local storage. This document provides a high-level introduction to the application's architecture, key components, and features.

## Purpose and Features

Kotatsu aims to provide a comprehensive manga reading experience with the following key capabilities:

* Reading manga from multiple online sources
* Downloading manga for offline reading
* Multiple reading modes (standard, webtoon, right-to-left, vertical)
* Tracking reading progress and history
* Favorites management with categories
* Customizable reading experience
* Chapter updates tracking and notifications
* Search and discovery features

## High-Level Architecture

Kotatsu follows modern Android development practices, implementing a multi-layer architecture that separates concerns between UI, domain logic, and data management.

## Core Components
### Application Structure
Kotatsu's main components are organized around key user flows:

`TODO diagram`

### Settings and Preferences
The app's settings are centralized in the `AppSettings` class, which provides access to user preferences through a SharedPreferences wrapper:

``` mermaid
classDiagram
  SharedPreferences <|-- AppSettings:uses
  AppSettings <|-- Activities:read/write
  AppSettings <|-- ViewModels:observe
  class SharedPreferences{
  }
  class Activities{
  }
  class ViewModels{
  }
  class AppSettings{
    -prefs: SharedPreferences
    +listMode: ListMode
    +theme: Int
    +colorScheme: ColorScheme
    +readerMode: ReaderMode
    +isReaderModeDetectionEnabled: Boolean
    +isTrackerEnabled: Boolean
    +isIncognitoModeEnabled: Boolean
    +subscribe()
    +unsubscribe(listener)  
  }
```

## Main Features
### Reader System
The reader is a core component of Kotatsu, providing multiple viewing modes and a customizable reading experience:

`TODO diagram`

The reader supports various features:

* Multiple reading modes (standard, right-to-left, vertical, webtoon)
* Customizable controls and gestures
* Page preloading
* Reading progress tracking
* Brightness and contrast adjustments
* Auto-scroll functionality

### Manga Sources System
Kotatsu can load manga from multiple sources, including online repositories and local storage:

`TODO diagram`

The source system allows:

* Browsing multiple manga sources
* Enabling/disabling sources
* Source-specific settings
* Automatic mirror switching for reliability
* Content filtering options

### User Interface
The app features a modern Material Design interface with:

* Bottom navigation for main sections
* List/grid view options for manga collections
* Dark and light themes with color schemes
* Support for multiple languages

## Technical Foundation
### Build System
Kotatsu uses Gradle for building, with dependency management through the Gradle Version Catalog:

| Component            | Description                          |
| -------------------- | ------------------------------------ |
| Build Tools          | Android Gradle Plugin 8.9.1          |
| Kotlin Version       | 2.1.20                               |
| Target SDK           | 35 (Android 15)                      |
| Minimum SDK          | 21 (Android 5.0)                     |
| Architecture         | MVVM with repositories               |
| Dependency Injection | Hilt                                 |
| Database             | Room                                 |
| Network              | OkHttp                               |
| Image Loading        | Coil 3                               |
| Async Processing     | Kotlin Coroutines                    |

### Dependencies and Libraries
Kotatsu integrates several key libraries:

* **AndroidX** components for UI and lifecycle management
* **Room** for local database storage
* **Hilt** for dependency injection
* **Coil** for efficient image loading
* **OkHttp** for network requests
* **Adapter** Delegates for RecyclerView implementation
* **WorkManager** for background tasks
* **Material Components** for UI elements

### Internationalization
The app supports multiple languages through Android's string resources system, with translations available for:

* English (default)
* Russian
* Spanish
* Turkish
* Belarusian
* Ukrainian
* French
* Italian
* Portuguese
* German
* Chinese
* And many others

## Summary
Kotatsu is a feature-rich manga reader application built with modern Android development practices. It offers a customizable reading experience, supports multiple manga sources, and provides convenient management features for manga collections. The app's architecture follows a clean separation of concerns between UI, domain logic, and data management, making it maintainable and extensible.

For more detailed information about specific aspects of the application, refer to the relevant sections in this wiki.