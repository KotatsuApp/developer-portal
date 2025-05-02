# Architecture

This page provides a high-level overview of kotatsu-dl's architecture, explaining the core components and how they interact to download manga from various sources. For detailed usage instructions, see [Usage Guide](../usage-guide/index.md), and for developer-focused implementation details, see [Developer Guide](../developer-guide/index.md).

## System Overview
Kotatsu-dl is a command-line manga downloader built using Kotlin. Its architecture follows a clean separation of concerns with distinct layers handling different aspects of the download process.

## Core Components
The system is organized around several key components that work together to download and organize manga content:

| Component | Purpose | Key Files |
| ------------ | ------------ | ---------- |
| Main Command | Entry point that parses CLI options and orchestrates the download process | Main.kt |
| Link Resolver | Resolves manga URLs to specific sources and retrieves manga metadata | Referenced in Main.kt |
| Manga Downloader | Coordinates downloading chapters and pages | Referenced in Main.kt |
| Local Manga Output | Abstract interface for saving manga to different formats | Referenced in diagrams |
| Manga Loader Context | Provides access to manga parsers | Referenced in Main.kt |

## Dependencies and Build System
Kotatsu-dl is built with Gradle using the Shadow plugin to create a self-contained executable JAR file.

The build system is configured to:

1. Compile Kotlin code
2. Package all dependencies into a single JAR file
3. Set the main class to `org.koitharu.kotatsu.dl.MainKt`
4. Minimize the JAR by removing unused classes

## Error Handling
The application implements robust error handling:

1. **Input Validation**: Command-line options are validated before processing
2. **Source Resolution**: Checks if the manga source is supported
3. **Manga Resolution**: Verifies the manga can be found
4. **Chapter Availability**: Ensures manga has downloadable chapters
5. **Download Errors**: Handles HTTP errors during download

Error messages are displayed to the user in a clear, human-readable format, often with color highlighting to improve readability.

## Concurrency Model
Kotatsu-dl employs Kotlin Coroutines for asynchronous operations, allowing efficient parallel downloads:

1. The `parallelism` parameter controls the number of concurrent downloads
2. The `throttle` option can slow down requests to avoid IP blocking
3. A dispatcher is used to execute downloads on background threads
4. Progress reporting is handled simultaneously with downloads

This approach provides efficient resource usage while maintaining responsiveness.

## Conclusion
Kotatsu-dl's architecture demonstrates a well-organized command-line application with clear separation of concerns:

1. **Command Interface Layer**: Handles user input and provides feedback
2. **Download Orchestration Layer**: Coordinates the download process
3. **Output Management Layer**: Handles saving content in various formats
4. **Parser Integration Layer**: Interfaces with manga sources

This modular design supports extensibility for new manga sources and output formats while maintaining a straightforward user experience.

For details on core components and their implementation, see [Core Components](core-components.md). For information about the download process, see [Download Process](download-process.md).

