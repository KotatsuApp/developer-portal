# Overview

Kotatsu Parsers is a library that provides a standardized interface for accessing manga from various web sources. It's designed to work in both JVM and Android applications, offering a unified API for fetching manga listings, details, chapters, and page images regardless of the source website structure.

This document provides a high-level overview of the Kotatsu Parsers architecture, its key components, and how they interact. For information about installation and usage, see [Installation and Usage](installation-and-usage.md).

## Purpose
The library solves several challenges in manga content aggregation:

1. Providing a consistent interface for diverse manga sources
2. Handling different website structures and APIs
3. Managing pagination, search, and filtering
4. Resolving direct image URLs for manga pages
4. Configuring parsers for different domains and user preferences

## Architecture Overview
The Kotatsu Parsers library is organized around a core `MangaParser` interface, with implementations for various manga websites. The library follows a modular architecture that separates the parser logic from the data models and network operations.

## Key Components
### MangaParser Interface
The [MangaParser](https://github.com/KotatsuApp/kotatsu-parsers/blob/master/src/main/kotlin/org/koitharu/kotatsu/parsers/MangaParser.kt) interface is the core of the library, defining the methods that all parsers must implement:

``` kotlin
interface MangaParser : Interceptor {
    val source: MangaParserSource
    val availableSortOrders: Set<SortOrder>
    val searchQueryCapabilities: MangaSearchQueryCapabilities
    val config: MangaSourceConfig
    val domain: String
    
    suspend fun getList(query: MangaSearchQuery): List<Manga>
    suspend fun getDetails(manga: Manga): Manga
    suspend fun getPages(chapter: MangaChapter): List<MangaPage>
    suspend fun getPageUrl(page: MangaPage): String
    suspend fun getFilterOptions(): MangaListFilterOptions
    suspend fun getFavicons(): Favicons
    fun onCreateConfig(keys: MutableCollection<ConfigKey<*>>)
    suspend fun getRelatedManga(seed: Manga): List<Manga>
}
```

This interface provides methods for:

* Listing manga with search and filter capabilities
* Retrieving detailed information about a manga
* Getting pages for a specific chapter
* Resolving the direct URL for a page image
* Getting filter options supported by the source

### Parser Base Classes
The library provides several abstract base classes that implement common parsing patterns:

1. **AbstractMangaParser**: A base implementation with common utilities
2. **LegacyPagedMangaParser**: For sites with page-based navigation
3. **LegacySinglePageMangaParser**: For sites with all content on a single page

The library provides a powerful search and filtering system with two approaches:

1. **Modern**: Using `MangaSearchQuery` with criteria like `Include`, `Exclude`, and `Match`
2. **Legacy**: Using `MangaListFilter` (now deprecated but still supported)

Each parser declares its search capabilities through `searchQueryCapabilities`, which defines what search fields and criteria types it supports.

### Configuration System
Parsers can be configured through a configuration system based on `ConfigKey`. Every parser must define a `configKeyDomain` which specifies the default domain and possible alternatives. This allows the application to handle domain changes without updating the library.

## Implementation Examples
The library includes parsers for many popular manga websites:

### API-Based Parsers
MangaDex is an example of a parser using a JSON API:

``` kotlin
@MangaSourceParser("MANGADEX", "MangaDex")
internal class MangaDexParser(context: MangaLoaderContext) : 
    AbstractMangaParser(context, MangaParserSource.MANGADEX) {
    
    override val configKeyDomain = ConfigKey.Domain("mangadex.org")
    
    // Implements methods to work with MangaDex API
}
```

### HTML-Based Parsers
Many parsers work with HTML content using JSoup:

``` kotlin
@MangaSourceParser("COMICK_FUN", "ComicK")
internal class ComickFunParser(context: MangaLoaderContext) :
    LegacyPagedMangaParser(context, MangaParserSource.COMICK_FUN, 20) {
    
    override val configKeyDomain = ConfigKey.Domain("comick.io")
    
    // Implements methods to parse HTML content
}
```

## Common Base Parsers
The library includes specialized base parsers for common website platforms:

1. **MadaraParser**: For sites using the Madara WordPress theme
2. **WpComicsParser**: For WordPress comic sites
3. **GroupleParser**: For Grouple-based sites
4. **MangaReaderParser**: For sites using MangaReader-like structures

This approach reduces code duplication and makes it easier to add support for new sites that use these common platforms.

## Conclusion
The Kotatsu Parsers library provides a robust and flexible framework for accessing manga from various online sources. Its architecture makes it easy to add new sources while maintaining a consistent interface for applications that use the library.

For information on how to include and use the library in your project, see [Installation and Usage](installation-and-usage.md). To learn about specific components in more detail, refer to the corresponding sections in this wiki.