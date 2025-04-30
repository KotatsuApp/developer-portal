# Adding a New Parser

This guide explains how to add a new manga source parser to the Kotatsu parsers library. It covers the process from evaluating the manga website to implementing and testing a parser. For information about parser architecture, see [Core Architecture](../core-architecture/index.md) and for code style conventions, see [Code Style and Guidelines](code-style-and-guidelines.md).

## Understanding the Parser Framework
Before adding a new parser, it's important to understand the basic architecture of the parsing system.

``` mermaid
classDiagram
  MangaParser <|-- AbstractMangaParser
  MangaParser <|-- LegacyMangaParser
  LegacyMangaParser <|-- LegacyPagedMangaParser
  LegacyPagedMangaParser <|-- TemplateParser
  TemplateParser <|-- SiteSpecificParser
  class SiteSpecificParser{
    <<concrete>>
  }
  class TemplateParser{
    <<abstract>>
  }
  class LegacyPagedMangaParser{
    <<abstract>>
  }
  class LegacyMangaParser{
    <<interface>>
  }
  class AbstractMangaParser{
    <<abstract>>
  }
  class MangaParser{
    <<interface>>
    +getList()
    +getDetails()
    +getPages()
    +getPageUrl()
    +getFilterOptions() 
  }
```

## Prerequisites
Before implementing a parser, you'll need:

1. Basic understanding of Kotlin
2. Knowledge of web scraping (JSoup) or JSON/GraphQL APIs
3. Familiarity with the website you're creating a parser for

## Evaluating a Manga Website
Start by investigating how the target website structures its content:

1. Does it use a common CMS like WordPress with a known theme (Madara, WpComics)?
2. Does it have a public API (REST, GraphQL)?
3. What elements need to be parsed (manga lists, chapter lists, pages)?

## Choosing a Base Parser
Based on your evaluation, select an appropriate base class:

| Website Type           | Recommended Base Class        | Notes                                              |
| ---------------------- | ----------------------------- | -------------------------------------------------- |
| Madara WordPress theme | `MadaraParser`                | Common for many manga sites                        |
| MangaReader-like sites | `MangaReaderParser`           | Sites with a similar structure to MangaReader      |
| Single page content    | `LegacySinglePageMangaParser` | Sites without pagination                           |
| Custom implementation  | `LegacyPagedMangaParser`      | For sites with pagination but no existing template |

## Implementation Steps
### 1. Create a New Parser Class
Create a new Kotlin file in the appropriate package under `org.koitharu.kotatsu.parsers.site`. The package structure follows `{site}.{language}` convention.

``` mermaid
flowchart TB
    id1[Choose base parser class] --> id2[Create new file in appropriate package] --> id3[Add @MangaSourceParser annotation] --> id4[Implement required methods] --> id5[Configure parser properties] --> id6[Test the parser]
```

### 2. Add Parser Annotation
Add the `@MangaSourceParser` annotation to your class with:

* Internal name (all caps)
* Display name
* Language code

For example:

``` kotlin
@MangaSourceParser("MANGASITE", "Manga Site", "en")
```

### 3. Extend Appropriate Base Class
Extend the appropriate base class and implement the constructor:

``` kotlin
internal class NewSiteParser(context: MangaLoaderContext) : 
    MadaraParser(context, MangaParserSource.NEWSITE, "mangasite.com")
```

Make sure to:

* Pass the `MangaLoaderContext` to the superclass
* Use the correct `MangaParserSource` enum value
* Specify the default domain

### 4. Configure Parser Properties
Override properties from the base class as needed:

``` kotlin
override val tagPrefix = "manga-genre/"
override val listUrl = "manga/"
override val datePattern = "MMMM d, yyyy"
```

The specific properties to override depend on the base class you're extending.


### 5. Implement Required Methods
If you're extending a template parser like `MadaraParser`, you might not need to override any methods. Otherwise, implement the necessary methods:

#### Key Methods to Implement
For custom parsers, you'll need to implement these methods:

| Method             | Purpose                                |
| ------------------ | -------------------------------------- |
| getListPage()      | Fetch a page of manga listing          |
| getDetails()       | Get detailed information about a manga |
| getPages()         | Get pages for a manga chapter          |
| getFilterOptions() | Return available filter options        |

### 6. Custom Selectors and Parsers
For HTML parsing, define CSS selectors for the elements you need to extract:

```
protected open val selectMangaList = ".manga-list .item"
protected open val selectChapter = "li.wp-manga-chapter"
protected open val selectPage = "div.page-break img"
```

Use these selectors in your parsing methods to extract the required information.

## Example Implementation
Here's an outline of implementing a parser for a Madara-based site:

1. Create a new file in the appropriate package
2. Add the `@MangaSourceParser` annotation
3. Extend `MadaraParser` and configure the domain
4. Override any necessary properties
5. If needed, override parsing methods

For more complex sites that don't use a common template, you'll need to implement custom parsing logic using JSoup or handle API responses.

## Testing Your Parser
### 1. Manual Testing
Test your parser with the following operations:

* Listing manga
* Searching
* Viewing manga details
* Reading chapters

### 2. Automated Testing
Before submitting, run the parser tests:

1. Modify the `MangaSources` annotation class to include your parser
2. Run the `MangaParserTest` test class
3. Verify the results with the test report

## Best Practices
### 1. Domain Handling
Never hardcode domains. Use `configKeyDomain` for the default domain and access it via the `domain` property:

``` kotlin
override val configKeyDomain = ConfigKey.Domain("mangasite.com")
```

### 2. Generating Unique IDs
Ensure all manga, chapter, and page IDs are unique and domain-independent:

``` kotlin
generateUid(relativeUrl)
```

### 3. Error Handling
Use extension functions like `selectFirstOrThrow` and `attrAsRelativeUrl` for better error handling:

``` kotlin
val a = element.selectFirstOrThrow("a")
val href = a.attrAsRelativeUrl("href")
```

### 4. Authentication Support
If your parser requires authentication, implement the `MangaParserAuthProvider` interface:

``` kotlin
internal class AuthenticatedParser(...) : BaseParser(...), MangaParserAuthProvider {
    override val authUrl: String get() = "https://$domain/login"
    override val isAuthorized: Boolean get() = /* check cookies */
    override suspend fun getUsername(): String { /* implementation */ }
}
```

## Advanced Topics
### Custom Network Interceptors
For sites with special requirements (CAPTCHA, CloudFlare protection), implement the `Interceptor` interface:

``` kotlin
internal class ProtectedSiteParser(...) : BaseParser(...), Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        // Custom handling of requests/responses
    }
}
```

## Conclusion
Adding a new parser involves understanding the target website, choosing an appropriate base class, and implementing the necessary methods. By following the patterns established in existing parsers, you can create a reliable parser that integrates well with the Kotatsu ecosystem.

For more assistance, refer to the [Core Architecture](../core-architecture/index.md) documentation or ask for help in the community channels mentioned in the CONTRIBUTING.md file.