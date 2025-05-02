# Download Process

This page details the internal download process of kotatsu-dl, explaining how the application fetches manga content from remote sources, handles errors, manages concurrent downloads, and processes the downloaded files. For information about output format handling, see [Output System](output-system.md).

## Overview
The download process is the core functionality of kotatsu-dl, responsible for efficiently retrieving manga content from various online sources while providing robust error handling, progress reporting, and performance optimization.

## MangaDownloader Class
The MangaDownloader class is the central component responsible for orchestrating the entire download process. It handles the preparation, execution, and cleanup phases of the manga download process.

### Key Parameters
| Parameter | Type | Description |
| --------------- | -------------- | -------------- |
| `context` | `MangaLoaderContext` | Provides HTTP client and parser access |
| `manga` | `Manga` | The manga to be downloaded |
| `chapters` | `List<MangaChapter>` | Available chapters for download |
| `destination` | `File?` | Target location for downloaded content |
| `chaptersRange` | `ChaptersRange` | Specifies which chapters to download |
| `format` | `DownloadFormat?` | Output format (ZIP, CBZ, directory) |
| `throttle` | `Boolean` | Whether to throttle requests |
| `verbose` | `Boolean` | Enable detailed console output |
| `parallelism` | `Int` | Maximum concurrent downloads |

## Key Implementation Details
### 1. Output Preparation
The download process starts by creating an appropriate output handler based on the destination and format:

``` kotlin
val output = LocalMangaOutput.create(destination ?: File("").absoluteFile, manga, format)
```

This factory call creates the correct implementation of `LocalMangaOutput` based on the output format (ZIP, CBZ, or directory).

### 2. Temporary Storage Management
The downloader creates a temporary directory to store downloaded files before they are processed and added to the final output:

``` kotlin
val tempDir = Files.createTempDirectory("kdl_").toFile()
```

This temporary directory is cleaned up in the `finally` block, ensuring resources are properly released even if exceptions occur.

### 3. Concurrent Page Downloads
The downloader uses Kotlin coroutines with a semaphore to limit concurrent downloads.

This approach allows for efficient parallel downloads while preventing server overload:

* A semaphore limits concurrent requests to the value of `parallelism`
* Each page download is launched as a separate coroutine
* All downloads for a chapter must complete before moving to the next chapter

### 4. Progress Tracking
The downloader uses a progress bar to provide visual feedback to the user:

* Initial setup with task name "Downloading"
* Updates the extra message with the current chapter name
* Estimates total work based on chapter pages
* Updates progress as each page is downloaded
* Displays "Finalizing..." during the output finishing phase

## Error Handling and Retry Mechanism
One of the key features of the download process is its robust error handling system implemented through the `runFailsafe` method.

The retry mechanism works as follows:

1. Attempts to run the provided block of code
2. Catches `IOException` (including `TooManyRequestExceptions`)
3. If retry count is not exhausted and retry delay is reasonable:
    * Decrements retry counter
    * Pauses the progress bar
    * Waits for the specified delay
    * Resumes the progress bar
    * Retries the operation
4. If retry conditions are not met, propagates the exception

Key constants:

* `MAX_FAILSAFE_ATTEMPTS = 2` - Maximum retry attempts
* `DOWNLOAD_ERROR_DELAY = 2_000L` - Default retry delay (2 seconds)
* `MAX_RETRY_DELAY = 7_200_000L` - Maximum retry delay (2 hours)

## Throttling Mechanism
To prevent triggering rate limits and to be respectful to the source servers, the download process implements a throttling mechanism.

The throttling implementation:

* Uses a global `DownloadSlowdownDispatcher` singleton
* Adds a configurable delay between requests to the same source
* Default delay is 500ms per source

## HTTP Request Configuration
The downloader configures HTTP requests with appropriate headers for image downloads:

``` kotlin
val request = Request.Builder()
    .url(url)
    .get()
    .header(CommonHeaders.ACCEPT, "image/webp,image/png;q=0.9,image/jpeg,*/*;q=0.8")
    .cacheControl(CommonHeaders.CACHE_CONTROL_NO_STORE)
    .tag(MangaSource::class.java, source)
    .build()
```

This configuration:

* Specifies acceptable image formats in order of preference
* Disables caching to ensure fresh content
* Tags the request with the manga source for tracking

## Integration with Other Components
The download process integrates with several other key components of the kotatsu-dl system:

1. Parser System:
    * Uses parser instances to fetch manga metadata
    * Retrieves chapter pages and page URLs
2. HTTP Client:
    * Handles network requests for downloads
    * Manages response processing
3. Output System:
    * Passes downloaded files to appropriate output handler
    * Interacts with output for managing chapters and metadata
4. Progress Visualization:
    * Updates progress bar with current status
    * Provides user feedback on download progress

## Error Handling Strategy
The download process implements a comprehensive error handling strategy:

1. Automatic Retries:
    * Failed network requests are retried up to MAX_FAILSAFE_ATTEMPTS times
    * Smart delay handling based on exception type
2. Rate Limit Handling:
    * Special handling for TooManyRequestExceptions
    * Respects server's requested retry delay
3. Resource Cleanup:
    * Uses finally block with NonCancellable context to ensure cleanup
    * Properly closes output resources
    * Deletes temporary files
4. User Interruption:
    * Catches CancellationException for clean handling of user interruptions
    * Displays appropriate message when interrupted

## Summary
The download process in kotatsu-dl is a robust, efficient system that handles the complexities of retrieving manga content from various online sources. Its key features include:

* Parallel downloading with controlled concurrency
* Comprehensive error handling with smart retries
* Progress tracking and user feedback
* Resource management and cleanup
* Integration with the parser and output systems

The process is designed to be resilient against network failures, respectful to source servers through throttling, and efficient in its use of system resources.

