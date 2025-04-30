# Core Architecture

This page provides an overview of the Kotatsu manga parser system's architecture, key components, and their relationships. It explains the foundational elements that enable the library to retrieve and process manga data from various online sources.

For detailed information about specific components, please refer to the dedicated pages: MangaParser Interface, Data Models, Search and Filter System, and Configuration System.

## System Overview
The Kotatsu parser system is designed around a core `MangaParser` interface that standardizes how manga content is retrieved from different websites. The architecture follows these key principles:

1. **Interface-based design**: All parsers implement the `MangaParser` interface
2. **Context-driven execution**: Parsers operate within a `MangaLoaderContext` that provides HTTP clients and configuration
3. **Hierarchical parser structure**: Abstract base parsers implement common functionality, with site-specific parsers inheriting from them
4. **Consistent data model**: Standardized models for manga, chapters, pages, and metadata

### Parser Hierarchy

``` mermaid
classDiagram
  MangaParser <|-- AbstractMangaParser
  MangaParser <|-- LegacyMangaParser
  LegacyMangaParser <|-- LegacyPagedMangaParser
  LegacyMangaParser <|-- LegacySinglePageMangaParser
  LegacyPagedMangaParser <|-- ConcreteParser2
  AbstractMangaParser <|-- ConcreteParser1
  class ConcreteParser1{
    <<example>>
    +MangaDexParser
  }
  class ConcreteParser2{
    <<example>>
    +ComickFunParser
  }
  class LegacyPagedMangaParser{
    <<abstract>>
    +pageSize: Int
    +getListPage(page: Int, order: SortOrder, filter: MangaListFilter)
  }
  class LegacySinglePageMangaParser{
    <<abstract>>
    +getList(query: MangaSearchQuery)
  }
  class LegacyMangaParser{
    <<interface>>
  }
  class AbstractMangaParser{
    <<abstract>>
    +context: MangaLoaderContext
    +source: MangaParserSource
  }
  class MangaParser{
    <<interface>>
    +source: MangaParserSource
    +availableSortOrders: Set<SortOrder>+searchQueryCapabilities: MangaSearchQueryCapabilities
    +domain: String
    +getList(query: MangaSearchQuery)
    +getDetails(manga: Manga)
    +getPages(chapter: MangaChapter)
    +getPageUrl(page: MangaPage)
    +getFilterOptions() 
  }
```

## Core Components
### 1. MangaParser Interface
The MangaParser interface is the cornerstone of the system, defining the contract that all parsers must implement to provide manga data:

``` mermaid
classDiagram
  class MangaParser{
    <<interface>>
    +source: MangaParserSource
    +availableSortOrders: Set<SortOrder>
    +searchQueryCapabilities: MangaSearchQueryCapabilities
    +config: MangaSourceConfig
    +configKeyDomain: ConfigKey.Domain
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

### 2. MangaLoaderContext
The MangaLoaderContext provides the environment for parsers to operate in:

``` mermaid
classDiagram
  class MangaLoaderContext{
    <<abstract>>
    +httpClient: OkHttpClient
    +cookieJar: CookieJar
    +newParserInstance(source: MangaParserSource) : MangaParser
    +newLinkResolver(link: HttpUrl) : LinkResolver
    +evaluateJs(script: String) : String?
    +getConfig(source: MangaSource) : MangaSourceConfig
    +getDefaultUserAgent() : String
    +redrawImageResponse(response: Response, redraw:(Bitmap) -> Bitmap) : Response
    +createBitmap(width: Int, height: Int) : Bitmap
  }
```

This context provides:

* HTTP client for web requests
* Cookie management
* JavaScript evaluation capabilities
* Configuration management
* Image processing utilities

### 3. Domain Model
The domain model represents the core data structures used throughout the system:

``` mermaid
classDiagram
  MangaChapter "0..*" <-- "1" Manga : contains
  MangaTag "0..*" <-- "1"  Manga : has
  MangaPage "0..*" <-- "1"  MangaChapter : contains
  class MangaPage{
    +id: Long
    +url: String
    +preview: String
    +source: MangaParserSource
  }
  class MangaTag{
    +title: String
    +key: String
    +source: MangaParserSource
  }
  class MangaChapter{
    +id: Long
    +url: String
    +title: String
    +number: Float
    +volume: Int
    +uploadDate: Long
    +scanlator: String
    +branch: String
    +source: MangaParserSource
  }
  class Manga{
    +id: Long
    +title: String
    +altTitles: Set<String>
    +url: String
    +publicUrl: String
    +coverUrl: String
    +tags: Set<MangaTag>
    +authors: Set<String>
    +state: MangaState
    +rating: Float
    +contentRating: ContentRating
    +chapters: List<MangaChapter>
    +source: MangaParserSource
  }
```

## Parser Implementation Pattern
The system follows a standard implementation pattern for manga parsers:

``` mermaid
sequenceDiagram
    "Client Code"->>"MangaParser": getList(query)
    "MangaParser"->>"MangaLoaderContext": Gets HTTP client
    "MangaParser"->>"MangaLoaderContext": Gets config
    "MangaParser"->>"WebClient": HTTP request
    "WebClient"->>"Manga Website": Sends HTTP request
    "Manga Website"->>"WebClient": Returns HTML/JSON
    "WebClient"->>"MangaParser": Returns response
    "MangaParser"->>"MangaParser": Parses data into Manga objects
    "MangaParser"->>"Client Code": Returns List<Manga>
    "Client Code"->>"MangaParser": getDetails(manga)
    "MangaParser"->>"WebClient": HTTP request
    "WebClient"->>"Manga Website": Sends HTTP request
    "Manga Website"->>"WebClient": Returns HTML/JSON
    "WebClient"->>"MangaParser": Returns response
    "MangaParser"->>"MangaParser": Parses details and chapters
    "MangaParser"->>"Client Code": Returns Manga with chapters
    "Client Code"->>"MangaParser": getPages(chapter)
    "MangaParser"->>"WebClient": HTTP request
    "WebClient"->>"Manga Website": Sends HTTP request
    "Manga Website"->>"WebClient": Returns HTML/JSON
    "WebClient"->>"MangaParser": Returns response
    "MangaParser"->>"MangaParser": Parses page URLs
    "MangaParser"->>"Client Code": Returns List<MangaPage>
```

## Search and Filter System
The search and filtering system allows clients to query manga sources with specific criteria:

``` mermaid
classDiagram
  QueryCriteria <|-- Match
  QueryCriteria <|-- Range
  QueryCriteria <|-- Exclude
  QueryCriteria <|-- Include
  MangaSearchQuery "1" *-- "0..*" QueryCriteria: contains
  MangaSearchQueryCapabilities "1" *-- "0..*" SearchCapability: defines
  class Include{
    +values: Set<T>
  }
  class Exclude{
    +values: Set<T>
  }
  class Range{
    +from: T
    +to: T
  }
  class Match{
    +value: T
  }
  class SearchCapability{
    +field: SearchableField
    +criteriaTypes: Set<Class>
    +isMultiple: Boolean
  }
  class QueryCriteria{
    <<interface>>
    +field: SearchableField
  }
  class MangaSearchQueryCapabilities{
    +capabilities: List<SearchCapability>
    +validate(query) : Boolean
  }
  class MangaSearchQuery{
    +criteria: List<QueryCriteria>
    +offset: Int
    +order: SortOrder?
  }
```

Key elements include:

* MangaSearchQuery: Represents a search query with criteria and pagination
* QueryCriteria: Interface for different types of search criteria
* MangaSearchQueryCapabilities: Defines what search capabilities a parser supports

## Configuration System
The configuration system allows parsers to be configured with domain settings, user agents, and other options:

``` mermaid
classDiagram
  MangaSourceConfig <-- MangaParser: uses
  `ConfigKey<T>` <-- MangaSourceConfig: contains
  `ConfigKey<T>` <|-- Domain
  `ConfigKey<T>` <|-- UserAgent
  class UserAgent{
    +key: "user_agent"
    +agent: String
  }
  class Domain{
    +key: "domain"
    +domains: List<String>
  }
  class `ConfigKey<T>`{
    <<abstract>>
    +key: String
    +defaultValue: T
  }
  class MangaSourceConfig{
    +get(key: ConfigKey<T>) : T?
    +set(key: ConfigKey<T>, value: T)
  }
  class MangaParser{
    <<interface>>
    +config: MangaSourceConfig
    +configKeyDomain: ConfigKey.Domain
    +onCreateConfig(keys: MutableCollection<ConfigKey>)
  }
```

Key components include:

* `ConfigKey`: Represents a configuration option with a key and default value
* `MangaSourceConfig`: Stores configuration for a manga source
* `onCreateConfig()`: Method called to register configuration keys

## Utility Functions
The system includes various utility functions to help with parsing:

| Function             | Purpose                                                                  |
| -------------------- | ------------------------------------------------------------------------ |
| `generateUid(url)`   | Generate a unique ID for manga, chapters, or pages                       |
| `oneOrThrowIfMany()` | Ensure only one filter option is selected when multiple aren't supported |
| `getDomain()`        | Get the domain for a parser with optional subdomain                      |
| `urlBuilder()`       | Create an HTTP URL builder with the parser's domain                      |

## Conclusion
The Kotatsu parsers architecture provides a flexible and extensible system for retrieving manga data from various sources. By standardizing the interface and providing common base implementations, new sources can be added easily while maintaining consistent behavior.

The architecture balances abstraction with practical implementation concerns, allowing developers to leverage shared functionality while accommodating the unique requirements of different manga websites.