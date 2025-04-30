# MangaParser Interface

The `MangaParser` interface is the central component of the Kotatsu parsers system, defining a standardized way to interact with various manga sources. This page documents the interface's core functionality, its methods, and how it fits into the broader architecture. For information about specific parser implementations, see Base Parser Implementations.

## Purpose and Role
The `MangaParser` interface acts as a contract that all manga source parsers must implement. It provides a unified API for:

* Searching and listing manga from a source
* Retrieving detailed information about manga
* Fetching chapter lists and page content
* Handling source-specific configuration
* Supporting various filtering and search capabilities

## Interface Definition
The MangaParser interface includes the following key properties and methods:

### Properties
| Property                       | Type                           | Purpose                                           |
| ------------------------------ | ------------------------------ | ------------------------------------------------- |
| `source`                       | `MangaParserSource`            | Identifies which manga source this parser is for  |
| `availableSortOrders`          | `Set<SortOrder>`               | Supported sorting options for manga lists         |
| `searchQueryCapabilities`      | `MangaSearchQueryCapabilities` | Defines what search criteria the source supports  |
| `config`                       | `MangaSourceConfig`            | Configuration for the parser instance             |
| `authorizationProvider`        | `MangaParserAuthProvider?`     | Provides authentication capabilities if supported |
| `configKeyDomain`              | `ConfigKey.Domain`             | Defines the domain(s) for the source              |
| `domain`                       | `String`                       | The current active domain for requests            |

### Core Methods
``` mermaid
classDiagram
  class MangaParser{
    <<interface>>
    +source: MangaParserSource
    +availableSortOrders: Set<SortOrder>
    +searchQueryCapabilities: MangaSearchQueryCapabilities
    +config: MangaSourceConfig
    +domain: String
    +getList(query: MangaSearchQuery) : List<Manga>
    +getDetails(manga: Manga) : Manga
    +getPages(chapter: MangaChapter) : List<MangaPage>
    +getPageUrl(page: MangaPage) : String
    +getFilterOptions() : MangaListFilterOptions
    +getFavicons() : Favicons
    +onCreateConfig(keys: MutableCollection<ConfigKey>)
    +getRelatedManga(seed: Manga) : List<Manga>
    +getRequestHeaders() : Headers
    +resolveLink(resolver: LinkResolver, link: HttpUrl) : Manga?
  }
```

#### Essential Methods
* **getList**: Retrieves a list of manga matching the provided search query

``` kotlin
suspend fun getList(query: MangaSearchQuery): List<Manga>
```

* **getDetails**: Retrieves complete information about a manga, including chapters

``` kotlin
suspend fun getDetails(manga: Manga): Manga
```

* **getPages**: Retrieves the list of pages for a manga chapter

``` kotlin
suspend fun getPages(chapter: MangaChapter): List<MangaPage>
```

* **getPageUrl**: Retrieves the direct URL to a page image

``` kotlin
suspend fun getPageUrl(page: MangaPage): String
```

#### Support Methods
* **getFilterOptions**: Retrieves available filtering options (tags, states, etc.)

``` kotlin
suspend fun getFilterOptions(): MangaListFilterOptions
```

* **getFavicons**: Retrieves website favicons for the source

``` kotlin
suspend fun getFavicons(): Favicons
```

* **onCreateConfig**: Initializes configuration options for the parser

``` kotlin
fun onCreateConfig(keys: MutableCollection<ConfigKey<*>>)
```

* **getRelatedManga**: Retrieves manga related to the provided manga

``` kotlin
suspend fun getRelatedManga(seed: Manga): List<Manga>
```

* **getRequestHeaders**: Retrieves HTTP headers for requests to the source

``` kotlin
fun getRequestHeaders(): Headers
```

* **resolveLink**: Resolves a web link to a manga object

``` kotlin
suspend fun resolveLink(resolver: LinkResolver, link: HttpUrl): Manga?
```

## Parser Implementation Patterns
Most manga parsers in the system follow one of these patterns:

1. Direct implementation of `MangaParser` interface (rare)
2. Extending `AbstractMangaParser` (modern approach)
3. Implementing `LegacyMangaParser` and extending either:
    * `LegacyPagedMangaParser` for paginated sources
    * `LegacySinglePageMangaParser` for single-page sources

The legacy parsers are gradually being replaced with modern implementations that use the new search API.

## Data Flow Through a Parser
The following sequence diagram illustrates the typical flow of data through a manga parser:

!!! danger
    TODO diagram

## Working with Data Models
The MangaParser interface works with several key data models:

### Manga
Represents a manga series with its metadata:

``` kotlin
data class Manga(
    val id: Long,           // Unique identifier
    val title: String,      // Primary title
    val altTitles: Set<String>, // Alternative titles
    val url: String,        // Relative URL (parser-specific)
    val publicUrl: String,  // Absolute URL for browsers
    val rating: Float,      // Normalized rating (0-1)
    val contentRating: ContentRating?, // Age rating
    val coverUrl: String?,  // URL to cover image
    val tags: Set<MangaTag>, // Tags/genres
    val state: MangaState?, // Publication status
    val authors: Set<String>, // Authors
    val largeCoverUrl: String? = null, // High-res cover
    val description: String? = null, // Description
    val chapters: List<MangaChapter>? = null, // Chapters
    val source: MangaSource // Source identifier
)
```

### MangaChapter
Represents a chapter of a manga:

``` kotlin
data class MangaChapter(
    val id: Long,          // Unique identifier
    val title: String?,    // Chapter title
    val number: Float,     // Chapter number
    val volume: Int,       // Volume number
    val url: String,       // Relative URL (parser-specific)
    val scanlator: String?, // Scanlation group
    val uploadDate: Long,  // Upload timestamp
    val branch: String?,   // Branch (e.g., language)
    val source: MangaSource // Source identifier
)
```

### MangaPage
Represents a page within a chapter:

``` kotlin
data class MangaPage(
    val id: Long,          // Unique identifier
    val url: String,       // Relative URL or resource identifier
    val preview: String?,  // Preview image URL (thumbnail)
    val source: MangaSource // Source identifier
)
```

### MangaTag
Represents a tag or genre:

```
data class MangaTag(
    val title: String,     // Display name
    val key: String,       // Identifier used for filtering
    val source: MangaSource // Source identifier
)
```

## Configuration System
The `MangaParser` interface includes a configuration system that allows parsers to:

1. Define their default domain and alternatives
2. Set content filtering options
3. Configure user agents and other HTTP parameters
4. Set source-specific options

### Key Configuration Methods
Each parser must implement:

* **configKeyDomain**: Define the domain(s) for the source

``` kotlin
override val configKeyDomain = ConfigKey.Domain("example.org", "example.com")
```

* **onCreateConfig**: Initialize configuration options

``` kotlin
override fun onCreateConfig(keys: MutableCollection<ConfigKey<*>>) {
    super.onCreateConfig(keys)
    keys.add(userAgentKey)
    keys.add(ConfigKey.ShowSuspiciousContent(false))
}
```

The `domain` property is then derived from the current value in the configuration.

## Search and Filter Capabilities
Modern parsers define their search capabilities using `MangaSearchQueryCapabilities`, which specifies:

1. Which fields can be searched (title, author, tags, etc.)
2. What types of criteria are supported (inclusion, exclusion, matching, ranges)
3. Whether multiple values are supported for each field

Example from MangaDex parser:

``` kotlin
override val searchQueryCapabilities: MangaSearchQueryCapabilities
    get() = MangaSearchQueryCapabilities(
        SearchCapability(
            field = TAG,
            criteriaTypes = setOf(Include::class, Exclude::class),
            isMultiple = true,
        ),
        SearchCapability(
            field = TITLE_NAME,
            criteriaTypes = setOf(Match::class),
            isMultiple = false,
        ),
        // Additional capabilities...
    )
```

## Implementing a New Parser
When implementing a new parser, you should:

1. Extend either AbstractMangaParser (recommended) or implement a legacy parser base class
2. Define the source's domain(s) and configuration
3. Implement the required methods for listing, retrieving details, and fetching pages
4. Define the source's search capabilities and filter options
5. Implement appropriate error handling

For detailed guidance on creating a new parser, see [Adding a New Parser](../contributing/adding-a-new-parser.md).

## Testing Parsers
The repository includes a comprehensive test suite for parsers in `MangaParserTest`. This verifies:

1. Basic list retrieval
2. Pagination handling
3. Search functionality
4. Tag filtering
5. Detail retrieval
6. Page retrieval
7. Domain configuration
8. Link resolution

## Legacy Support
The interface includes deprecated methods to support legacy code:

``` kotlin
@Deprecated("Use getList(query: MangaSearchQuery) instead")
suspend fun getList(offset: Int, order: SortOrder, filter: MangaListFilter): List<Manga> {
    return getList(convertToMangaSearchQuery(offset, order, filter))
}

@Deprecated("Please check searchQueryCapabilities")
val filterCapabilities: MangaListFilterCapabilities
    get() = searchQueryCapabilities.toMangaListFilterCapabilities()
```

These provide backward compatibility while encouraging migration to the new search API.

