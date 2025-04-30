# Overview

This document provides a technical overview of kotatsu-dl, a cross-platform manga downloader command-line application that supports a wide range of manga sources. It explains the system's core architecture, components, and general workflow.

For detailed installation instructions, see Installation & Setup. For usage examples and command line options, see Usage Guide.

## What is kotatsu-dl?
kotatsu-dl is a command-line tool written in Kotlin that allows users to download manga from various online sources into local archives (CBZ, ZIP) or directory structures. It provides a simple interface for specifying manga URLs, selecting chapters to download, and configuring output formats.

Key features include:

* Support for numerous manga sources (websites)
* Multiple output formats (CBZ, ZIP, directory)
* Parallel downloading capabilities
* Throttling to avoid IP blocking
* Interactive chapter selection

## System Architecture
kotatsu-dl follows a modular architecture with clear separation of concerns. The system consists of these main components:

1. **Command Line Interface**: Handles user input and provides feedback
2. **Link Resolver**: Analyzes and validates manga URLs
3. **Manga Downloader**: Orchestrates the download process
4. **Output Manager**: Handles saving content in different formats

## Workflow
The typical workflow of kotatsu-dl follows these steps:

1. User provides a manga URL and optional parameters via command line
2. The system resolves the source and validates the URL
3. Manga metadata and chapter information are fetched
4. User selects which chapters to download (interactively or via parameters)
5. The system downloads chapter content in parallel (with optional throttling)
6. Content is processed and saved in the specified output format

## Integration with kotatsu-parsers
kotatsu-dl leverages the kotatsu-parsers library to support a wide range of manga sources. This integration allows the application to:

1. Interpret URLs from various manga websites
2. Extract metadata consistently across sources
3. Navigate chapter structures regardless of source format
4. Handle website-specific quirks through specialized parsers

The `MangaLoaderContextImpl` class initializes the parsing system and provides access to the appropriate parser for each supported source.

## Output Formats
kotatsu-dl supports multiple output formats to accommodate different user preferences:

| Format | Description                                   | Best Use Case                      |
| ------ | --------------------------------------------- | ---------------------------------- |
| CBZ    | Comic Book ZIP - standard comic reader format | Reading in comic/manga reader apps |
| ZIP    | Standard ZIP archive                          | General storage and distribution   |
| DIR    | Plain directory structure                     | Editing or manual organization     |

## Technologies and Dependencies
kotatsu-dl is built with Kotlin and relies on several key dependencies:

| Dependency        | Purpose                                         |
| ----------------- | ----------------------------------------------- |
| Kotlin Coroutines | Asynchronous programming and parallel downloads |
| Clikt             | Command-line interface framework                |
| kotatsu-parsers   | Manga source parsing capabilities               |
| OkHttp            | HTTP client for network requests                |
| Shadow            | JAR packaging for distribution                  |

## Build and Distribution
kotatsu-dl is packaged using the Shadow plugin to create a self-contained executable JAR file. This approach ensures that all dependencies are included in a single file, making distribution and execution straightforward across platforms.