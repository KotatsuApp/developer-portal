# Core Components

This document provides a detailed description of the core components that make up the kotatsu-dl manga downloader. We'll examine the main architectural elements, their responsibilities, and how they interact with each other. For information about the actual download process, see [Download Process](download-process.md), and for details on how outputs are handled, see [Output System](output-system.md).

## Overview of Core Components
Kotatsu-dl is structured around several key components that work together to provide manga downloading functionality.

## Command Layer
The command layer is responsible for handling user input through the command line interface and coordinating the download process.

### AppCommand
`AppCommand` is an abstract base class that provides common functionality for all commands in the application:

* Sets up the command-line environment
* Handles error processing and reporting
* Implements a consistent approach to command execution

!!! abstract "Key features"
    * Extends CoreSuspendingCliktCommand from the Clikt library
    * Provides error handling for common exceptions
    * Abstracts environment interactions like file reading and process termination
    * Defines an abstract invoke() method that subclasses must implement

### Main Command
The Main class is the primary command implementation and entry point for the application:

* Processes command-line arguments and options
* Coordinates the entire download workflow
* Handles user interaction for chapter selection

!!! abstract "Key responsibilities"
    * Accepts and validates the manga URL and download parameters
    * Resolves the manga link to identify the source and manga details
    * Facilitates chapter selection through user interaction
    * Creates and configures the MangaDownloader to perform the actual download
    * Reports on the download progress and results

## Download Layer
The download layer handles the core functionality of retrieving manga content from various sources.

### MangaDownloader
`MangaDownloader` is the central component responsible for downloading manga chapters:

* Controls the download process for manga covers and chapter pages
* Manages parallelism and throttling
* Coordinates with the output system to save downloaded content

!!! abstract "Key features"
    * Configurable parallelism to control concurrent downloads
    * Throttling capability to prevent IP blocking
    * Progress reporting for verbose output
    * Integration with LocalMangaOutput to save downloaded content

### LinkResolver
The `LinkResolver` component is responsible for:

* Parsing manga URLs to identify the source
* Retrieving manga metadata
* Converting generic URLs into structured manga information

!!! abstract "Key functionality"
    * URL pattern matching to identify the manga source
    * Validation of manga availability
    * Extraction of manga metadata (title, cover, chapters)

## Output Layer
The output layer handles the storage and organization of downloaded manga content.

### LocalMangaOutput
LocalMangaOutput is an abstract base class that defines the interface for different output formats.

!!! abstract "Key operations"
    * Adding cover images
    * Adding chapter pages
    * Finalizing chapters
    * Completing and cleaning up the output

### Output Implementations
Two main implementations exist:

1. LocalMangaZipOutput: Creates compressed archive files (ZIP/CBZ format)
2. LocalMangaDirOutput: Creates an organized directory structure with manga chapters and pages

The actual implementation is selected based on:

* User-specified format option
* Destination file path extension
* Number of chapters in the manga

## Integration with Parser Layer
The kotatsu-dl application integrates with the kotatsu-parsers library to support various manga sources.

### MangaLoaderContext
`MangaLoaderContext` serves as the bridge between kotatsu-dl and the parsers:

* Provides HTTP client configuration
* Creates parsers for specific manga sources
* Handles source-specific authentication and requests

### MangaParserSource
`MangaParserSource` is an enumeration of supported manga sources:

* Defines the available manga sources
* Provides metadata about each source (name, URL patterns)
* Enables source-specific parsing logic

## Summary
The core components of kotatsu-dl work together to provide a robust manga downloading solution:

1. The **Command Layer** (`AppCommand`, `Main`) processes user input and orchestrates the download workflow
2. The **Download Layer** (`MangaDownloader`, `LinkResolver`) handles the actual retrieval of manga content
3. The **Output Layer** (`LocalMangaOutput` implementations) manages storage of downloaded content
4. The **Parser Layer** integration enables support for various manga sources

These components are designed with clear responsibilities and interfaces, allowing for flexibility in output formats and easy addition of new manga sources.