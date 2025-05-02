# Testing Framework

The Kotatsu Parsers library includes a comprehensive testing framework designed to validate the functionality and reliability of manga source parsers. This document describes the testing structure, organization, and utilities that help ensure parsers correctly implement the expected behavior.

## Overview
The testing framework in Kotatsu Parsers aims to provide consistent validation for all parser implementations by running standardized tests against each parser. This ensures that parsers handle manga listings, details, chapters, pages, and other functionalities correctly according to the requirements of the `MangaParser` interface.

## Test Structure
The framework follows a parameterized testing approach, where each test case is executed against all available source parsers.

### Main Test Class
The central component is the `MangaParserTest` class, which contains a collection of standardized test methods that validate different aspects of parser functionality.

### Test Context
Tests run using `MangaLoaderContextMock`, which simulates the environment parsers operate in. This context provides:

* Factory methods to create parser instances
* HTTP request handling for testing network operations
* Link resolution capabilities

### Test Annotations
The test methods use JUnit's `@ParameterizedTest` combined with a custom `@MangaSources` annotation, which supplies all available manga sources as parameters. This ensures each test is run for every parser implementation.

## Test Coverage
The testing framework provides comprehensive coverage of parser functionality:

### Manga Listing and Pagination
Tests verify that parsers can retrieve manga lists and handle pagination correctly.

The framework verifies:

* Lists contain valid manga entries
* Pagination returns different content across pages
* Parsers respect page size limits

### Search Functionality
Tests search functionality by:

* Finding manga by title
* Testing tag filtering
* Verifying multiple tag support
* Testing language/locale filtering

### Manga Details and Chapters
Tests validate that parsers can:

* Retrieve complete manga details
* Fetch chapter information
* Ensure chapters have proper metadata (upload dates, numbers, etc.)
* Validate cover images

### Page Retrieval and Image Validation
Tests verify:

* Parsers can retrieve chapter pages
* Page URLs are valid and accessible
* Image content is properly loaded

### Additional Validations
The framework also tests:

* Favicon retrieval
* Domain configuration
* Public URL resolution
* Authentication (disabled by default)

## Test Assertions and Verification
The framework uses a robust set of assertions to validate parser behavior:

### Manga List Validation
The `checkMangaList` method ensures that manga lists:

* Are not empty
* Contain distinct manga IDs
* Have valid URLs
* Include proper titles
* Have accessible cover images

``` kotlin
private suspend fun checkMangaList(list: List<Manga>, cause: String) {
    assert(list.isNotEmpty()) { "Manga list for '$cause' is empty" }
    assert(list.isDistinctBy { it.id }) { "Manga list for '$cause' contains duplicated ids" }
    // ... additional checks
}
```

### Image Request Validation
The `checkImageRequest` method verifies image URLs:

* Can be successfully requested
* Return proper image MIME types

``` kotlin
private suspend fun checkImageRequest(url: String?, source: MangaSource) {
    if (url == null) {
        return
    }
    context.doRequest(url, source).use {
        assert(it.isSuccessful) { "Request failed: ${it.code}(${it.message}): $url" }
        assert(it.mimeType?.startsWith("image/") == true) {
            "Wrong response mime type: ${it.mimeType}"
        }
    }
}
```

## Running Tests
The test class is designed to run tests for all parser implementations. Every test method is executed against each manga source with a specified timeout (2 minutes by default).

``` kotlin
@ParameterizedTest(name = "{index}|list|{0}")
@MangaSources
fun list(source: MangaParserSource) = runTest(timeout = timeout) {
    val parser = context.newParserInstance(source)
    val list = parser.getList(MangaSearchQuery.Builder().build())
    checkMangaList(list, "list")
    assert(list.all { it.source == source })
}
```

## Extending the Testing Framework
When adding a new parser implementation:

1. The parser must implement the `MangaParser` interface or extend one of the abstract base parsers
2. It will automatically be included in tests via the `@MangaSources` annotation
3. No additional test code is needed - all standard tests will run against the new parser

## Testing Utilities
The testing framework includes several utilities:

* String validation helpers for checking URLs and formatting
* Collection extensions for distinct element validation
* Chapter key generation for uniqueness testing

### Integration with Build System
The test framework is integrated with the build system to collect and report test results, with test case models defined in the `buildSrc` directory.

## Conclusion
The Kotatsu Parsers testing framework provides comprehensive validation of parser implementations through automated, parameterized tests. This ensures that all parsers meet the required standards for manga retrieval, search, and content access.

By standardizing the testing approach, the framework makes it easy to verify new parser implementations and maintain existing ones, ensuring consistency across the entire parser ecosystem.