---
title: Domain Layer
description: The Domain Layer in Kotatsu implements the core business logic of the manga reader application.
---

# Domain Layer

The Domain Layer in Kotatsu implements the core business logic of the manga reader application. It sits between the UI/presentation layer and the data layer, containing the application's essential functionality like page loading, manga tracking, suggestions, and error handling. This layer is responsible for orchestrating data operations and implementing business rules independent of how data is stored or presented to users.

For information about the Data Layer that provides data access and persistence, see [Data Layer](data-layer.md).

## Core Components

The Domain Layer consists of several key components that handle specific aspects of the application's business logic:

`TODO diagram`

## Page Loading System

The Page Loading System, centered around the `PageLoader` class, is responsible for fetching, caching, and processing manga pages for display in the reader.

### PageLoader Functionality

`TODO diagram`

The `PageLoader` class provides several key functions:

1. **Page Loading**: The `loadPageAsync` and `loadPage` methods fetch manga pages from various sources (network, local files, ZIP archives) and return `Uri` references to the loaded content.

2. **Prefetching**: The `prefetch` method queues pages to be loaded ahead of time, improving reading experience by reducing wait times.

3. **Image Processing**: The `convertBitmap` method handles converting images between formats, while `getTrimmedBounds` detects page edges to remove unnecessary whitespace.

4. **Caching**: Works with `PagesCache` to store and retrieve manga pages, minimizing network usage and improving performance.

The `PageLoader` employs concurrency control mechanisms:

* `Semaphore` to limit parallel downloads
* `Mutex` for thread-safe conversion operations
* Background coroutine scope for asynchronous operations

## Tracking System

The Tracking System monitors manga for new chapters and notifies users of updates. It's primarily implemented through the TrackWorker and related components.

`TODO diagram`

### Key Components

1. **TrackWorker**: A `CoroutineWorker` that periodically checks for manga updates in the background. It handles worker scheduling, notification management, and download operations.

2. **CheckNewChaptersUseCase**: Contains the core logic for detecting new chapters by comparing the manga's current state with previously tracked information.

3. **TrackingRepository**: Manages persistent storage of tracking data including chapter information and update history.

4. **MangaUpdates**: A sealed interface with two implementations:
    * `Success`: Contains information about new chapters
    * `Failure`: Represents error states with retry logic

### Update Detection Process

The tracking system follows these steps:

1. Retrieve tracked manga from the database
2. For each manga, fetch the latest details from its source
3. Compare chapter IDs and timestamps to identify new chapters
4. Save update information to the database
5. Display notifications for new chapters
6. Optionally trigger downloads for new chapters

## Suggestions System

The Suggestions System analyzes user reading habits and recommends similar manga. It's implemented via the `SuggestionsWorker` class.

`TODO diagram`

### Recommendation Algorithm

The suggestions system uses a sophisticated algorithm to find manga similar to what users enjoy:

1. **Data Collection**: Analyzes the user's reading history and favorites
2. **Tag Extraction**: Identifies common tags from the user's manga
3. **Source Selection**: Queries enabled manga sources (or all sources if configured)
4. **Relevance Calculation**: Computes similarity scores based on tag matching
5. **Filtering**: Removes NSFW content if appropriate, applies blacklist rules
6. **Result Storage**: Saves suggestions for display in the UI
7. **Notification**: Sends notifications about selected recommendations

The relevance score is calculated using a weighted algorithm that prioritizes tags that appear frequently in the user's reading history.

## Error Handling

The Domain Layer includes comprehensive error handling mechanisms to manage various failure scenarios.

`TODO diagram`

### Key Features

1. **User-Friendly Messages**: The `getDisplayMessage` method transforms technical errors into readable messages.
2. **Visual Indicators**: The `getDisplayIcon` method provides appropriate icons for error states.
3. **Cloudflare Protection Handling**: The `CaptchaNotifier` helps users solve Cloudflare CAPTCHA challenges when accessing protected sites.
4. **Error Classification**: Methods like `isReportable()` and `isNetworkError()` categorize errors for appropriate handling.
5. **Exception Recovery**: Various mechanisms to attempt recovery from non-fatal errors.

## Worker Processes

The Domain Layer uses several background worker processes to perform tasks while the app is in the background.

`TODO diagram`

### Worker Characteristics
1. **Scheduling**: Workers use both periodic and one-time scheduling depending on the task.
2. **Constraints**: Workers define network, battery, and storage constraints to run under appropriate conditions.
3. **Foreground Operation**: For long-running tasks, workers can operate as foreground services with notifications.
4. **Retry Mechanisms**: Failed operations have configurable backoff policies.
5. **User Control**: Workers respect user settings for Wi-Fi-only operation, frequency, and enabled features.

## Storage Management

The Domain Layer interacts with various storage systems for manga files, pages cache, and temporary data.

`TODO diagram`

### Storage Features

1. **Directory Management**: The system manages application-specific directories and user-specified locations for manga storage.
2. **Caching**: The `PagesCache` provides efficient storage and retrieval of manga pages, with automatic size management.
3. **URI Handling**: Special URI schemes (like `file+zip://`) allow unified access to different storage types.
4. **Storage Analysis**: Functions to compute available space, usage statistics, and storage capabilities.
5. **Import/Export**: Support for importing manga from external sources and ZIP/CBZ archives.

## Use Cases

The Domain Layer contains numerous use cases that encapsulate specific business operations. Here are some key examples:

| Use Case                  | Purpose                        | Key Operations                                    |
| ------------------------- | ------------------------------ | ------------------------------------------------- |
| `CheckNewChaptersUseCase` | Detect new manga chapters      | Compare chapter lists, identify updates           |
| `DetectReaderModeUseCase` | Determine optimal reader mode  | Analyze page dimensions, detect webtoon format    |
| `MigrateUseCase`          | Transfer data between manga    | Copy favorites, history, and reading progress     |
| `DetailsLoadUseCase`      | Load manga details             | Fetch metadata, chapters, and descriptions        |

### Domain Models
The Domain Layer defines several important model classes that represent business entities:

`TODO diagram`

## Interaction with Other Layers
The Domain Layer orchestrates operations between the UI and Data layers:

`TODO diagram`

### Key Interactions
1. **View Models to Domain**: UI components interact with the domain layer through ViewModels, which invoke use cases and domain services.
2. **Domain to Data**: Domain components access data through repositories, which abstract the data sources.
3. **Background Processing**: Workers operate independently of UI components, coordinating with repositories for data access.
4. **Event Notifications**: The domain layer communicates state changes back to the UI layer via flows and callbacks.