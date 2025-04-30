---
title: Code Style and Guidelines
description: This document outlines the coding conventions, best practices, and technical requirements
---

# Code Style and Guidelines

This document outlines the coding conventions, best practices, and technical requirements for contributing to the Kotatsu parsers library. Following these guidelines ensures consistency across the codebase and helps maintain a high standard of quality. For information about adding a new parser, see [Adding a New Parser](adding-a-new-parser.md).

## 1. Project Structure
### 1.1 Package Organization
The Kotatsu parsers codebase follows a structured package organization:

```
org.koitharu.kotatsu.parsers
├── core/           - Abstract base parser implementations
├── model/          - Data models (Manga, MangaChapter, etc.)
├── site/           - Site-specific parser implementations
│   ├── all/        - Multi-language sites
│   ├── en/         - English sites
│   ├── ru/         - Russian sites
│   ├── ...
├── util/           - Utility classes and extensions
└── config/         - Parser configuration
```

### 1.2 Parser Class Naming
Parser classes should follow a consistent naming pattern:

* Use the site name followed by "Parser" (e.g., `MangaDexParser`, `ComickFunParser`)
* For sites based on a common engine, extend the appropriate base parser (e.g., `MadaraParser`, `WpComicsParser`)

## 2. Code Conventions
### 2.1 Parser Class Structure
Each parser must:

1. Be annotated with @MangaSourceParser specifying:
    * Source identifier (enum constant name)
    * Display name
    * Optional language code
2. Extend the appropriate base parser class
3. Implement required methods
4. Define the `configKeyDomain` field

Example parser class structure:

``` kotlin
@MangaSourceParser("SOURCE_NAME", "Source Display Name", "en")
internal class ExampleParser(context: MangaLoaderContext) : 
    AbstractMangaParser(context, MangaParserSource.SOURCE_NAME) {
    
    override val configKeyDomain = ConfigKey.Domain("example.com")
    override val availableSortOrders = EnumSet.of(SortOrder.UPDATED)
    
    // Implementation methods...
}
```

## 3. Naming Conventions
### 3.1 General Naming
* Classes: PascalCase (e.g., `MangaParser`, `MangaDexParser`)
* Methods/functions: camelCase (e.g., `getList`, `fetchAvailableTags`)
* Constants: UPPER_SNAKE_CASE (e.g., `RATING_UNKNOWN`, `YEAR_MIN`)
* Properties/variables: camelCase (e.g., `availableSortOrders`, `tagsMap`)
### 3.2 Constants and Magic Numbers
* Never use magic numbers in code
* Define constants at the file or class level
* Use meaningful and descriptive constant names

!!! failure "Bad"
    ``` kotlin
    if (rating > -1f) { ... }
    ```

!!! success "Good"
    ``` kotlin
    if (rating > RATING_UNKNOWN) { ... }
    ```

## 4. Domain Handling
### 4.1 Domain Configuration
* Never hardcode domains in requests
* Always use the `configKeyDomain` field to specify the default domain
* Use the `domain` property to get the current domain value
* Use `urlBuilder()` to create HTTP URLs

``` kotlin
// Incorrect
val url = "https://example.com/manga/$mangaId"

// Correct
val url = "https://api.$domain/manga/$mangaId"
// Or
val url = urlBuilder()
    .addPathSegment("manga")
    .addPathSegment(mangaId)
    .build()
```

## 5. ID Generation
### 5.1 Unique IDs
* All manga, chapters, and pages must have unique IDs
* Use `generateUid()` to create consistent IDs
* IDs must be domain-independent (work across domain changes)
* Use relative URLs or internal IDs as input to `generateUid()`

``` kotlin
// Incorrect
manga.id = url.hashCode().toLong()

// Correct
manga.id = generateUid(mangaSlug)
```

## 6. Error Handling
### 6.1 Error Messages
* Use constants from `ErrorMessages` for common error situations
* Provide meaningful error messages that help diagnose the issue
* Use assertions (`assert`) to verify optional fields during development
### 6.2 Exception Handling
* Throw `ParseException` for parsing errors
* Use helper methods like `oneOrThrowIfMany()` for filter validation
* Document behavior that may result in exceptions

## 7. Search and Filter System
### 7.1 Search Capabilities
* Define `searchQueryCapabilities` to specify supported search features
* Use `MangaSearchQueryCapabilities` with appropriate `SearchCapability` entries
* Follow the capabilities pattern when implementing `getList`

``` kotlin
override val searchQueryCapabilities: MangaSearchQueryCapabilities
    get() = MangaSearchQueryCapabilities(
        SearchCapability(
            field = TAG,
            criteriaTypes = setOf(Include::class, Exclude::class),
            isMultiple = true,
        ),
        // Other capabilities...
    )
```

### 7.2 Filter Implementation
* Validate filter parameters before using them
* For single-selection filters, use `oneOrThrowIfMany()`
* Properly handle empty filters

## 8. Content Formatting
### 8.1 Manga Metadata
* Tag titles should be capitalized
* Empty strings should not be used for tags or titles
* Parse dates using appropriate date formats

``` kotlin
// Incorrect
MangaTag(title = "action", key = "action", source = source)

// Correct  
MangaTag(
    title = "Action".toTitleCase(Locale.ENGLISH),
    key = "action",
    source = source
)
```

### 8.2 URL Formatting
* Manga URLs should be relative (no domain)
* Public URLs should be absolute
* Cover URLs should be absolute
* Page URLs can be temporary in `getPages()` but must resolve to direct image URLs in `getPageUrl()`

## 9. Testing Requirements
All parsers must pass the following tests:

| Test                | Description                                                |
| ------------------- | ---------------------------------------------------------- |
| `list`              | Retrieve a non-empty list of manga                         |
| `pagination`        | Verify pagination works with non-intersecting pages        |
| `searchByTitleName` | Search for a specific manga by title                       |
| `tags`              | Verify tags are correctly retrieved and used for filtering |
| `tagsMultiple`      | Test multiple tag filtering if supported                   |
| `locale`            | Test locale filtering if supported                         |
| `details`           | Fetch and verify manga details                             |
| `pages`             | Retrieve and verify chapter pages                          |
| `favicon`           | Verify favicon retrieval                                   |
| `domain`            | Test domain resolution                                     |
| `link`              | Test public link resolution                                |

## 10. Documentation
### 10.1 Code Documentation
* Add KDoc comments for public classes and methods
* Document non-obvious behavior
* Use `@Deprecated` with migration instructions for deprecated methods

``` kotlin
/**
 * Parse details for [Manga]: chapters list, description, large cover, etc.
 * Must return the same manga, may change any fields except id, url and source
 * @see Manga.copy
 */
public suspend fun getDetails(manga: Manga): Manga
```

### 10.2 Special Annotations
* Use `@MangaSourceParser` to define parser metadata
* Use `@Broken` to mark non-working parsers
* Use `@InternalParsersApi` for API not intended for public use

## 11. Best Practices
### 11.1 Performance Considerations
* Use coroutine scope for parallel operations
* Implement pagination efficiently
* Cache frequently accessed data with `suspendLazy`
* Use appropriate data structures (e.g., `EnumSet` for enum collections)

### 11.2 Code Organization
* Group related functionality
* Extract reusable parsing logic
* Use extension functions for common operations
* Prefer composition over inheritance when appropriate

## 12. Configuration System
* Implement `onCreateConfig()` to register parser-specific configuration keys
* Use configuration values via the `config` property
* Define appropriate default values
* Document configuration options