# Contributing

This document provides comprehensive guidelines for contributing to the Kotatsu Parsers library. It covers the entire process from setting up your development environment to submitting pull requests. For specific guidance on adding a new parser, see [Adding a New Parser](adding-a-new-parser.md), and for code style information, see [Code Style and Guidelines](code-style-and-guidelines.md).

## Prerequisites
Before starting your contribution to Kotatsu Parsers, ensure you have the following knowledge and tools:

### Required Knowledge
* **Kotlin programming language**
* **Web scraping techniques** (using JSoup) or **JSON/GraphQL API integration**
* Basic understanding of **Android development** (helpful but not strictly required)

### Development Tools
* IntelliJ IDEA (Community Edition is sufficient) or Android Studio
* JDK 8 or later
* Git

## Projct Setup

1. Fork the repository from https://github.com/KotatsuApp/kotatsu-parsers
2. Clone your forked repository locally
3. Import the project into your IDE as a Gradle project
4. Run Gradle sync to download all dependencies

## Parser Architecture
Kotatsu Parsers uses a hierarchical architecture to organize different types of parsers. Understanding this structure is essential before creating your own parser.

!!! danger
    TODO diagram

## Development Workflow
The following diagram illustrates the typical workflow for contributing a new parser to the Kotatsu Parsers project:

!!! danger
    TODO diagram

## Creating a Parser

Each parser in Kotatsu Parsers is a single class that implements the necessary functionality to access manga from a specific website.

### Basic Requirements
1. Parser Class Structure:
    * Must extend `MangaParser` or one of its subclasses
    * Must have exactly one primary constructor parameter of type `MangaLoaderContext`
    * Must have `@MangaSourceParser` annotation specifying name, title, and language
2. Key Implementation Details:
    * Do not hardcode domains - use configKeyDomain and the domain property
    * Generate unique IDs using the generateUid function
    * Define at least one value in availableSortOrders
    * Implement required methods based on the parent class

### Example Parser Structure

``` kotlin
@MangaSourceParser("SOURCE_NAME", "Source Title", "language-code")
internal class YourParser(context: MangaLoaderContext) :
    MadaraParser(context, MangaParserSource.SOURCE_NAME, "example.com") {
    
    // Override necessary methods and properties
    override val tagPrefix = "manga-tag/"
    
    // Implement custom logic if needed
}
```

## Choosing the Right Base Class
Select the appropriate base class based on the website's structure:

| Base Class              | When to Use                                       |
| ----------------------- | ------------------------------------------------- |
| `MadaraParser`          | For sites using the Madara WordPress theme        |
| `WpComicsParser`        | For WordPress comics sites                        |
| `GroupleParser`         | For Grouple-based sites                           |
| `PagedMangaParser`      | For sites using page-based pagination             |
| `SinglePageMangaParser` | For sites with no pagination                      |
| Custom implementation   | When no existing parser fits the site's structure |

## Testing Your Parser
Proper testing is crucial before submitting your contribution:

### Running Tests Locally
1. Temporarily modify the `MangaSources` annotation class:
    * Specify your parser name(s)
    * Change mode to `EnumSource.Mode.INCLUDE`
2. Run the unit tests:
``` bash
./gradlew :test --tests "org.koitharu.kotatsu.parsers.MangaParserTest"
```
3. Generate a test report (optional):
``` bash
./gradlew generateTestsReport
```

### Automated Testing in CI
When submitting a pull request, GitHub Actions will automatically run tests for your parser if it modifies files in the parsers directory.

## Submitting Your Contribution
When your parser is ready for submission:

1. Ensure all tests pass locally
2. Move your parser code to the appropriate package in the `kotatsu-parsers` project
3. Create a pull request with a clear description of your changes
4. Respond to any feedback during the review process

## Getting Help
If you encounter issues or have questions during development, reach out to the community:

* [Telegram chat](https://t.me/kotatsuapp)
* [Discord server](https://discord.gg/NNJ5RgVBC5)

## Best Practices
1. **Study existing parsers** for similar websites before starting your own
2. **Use utility extensions** from the `util` package rather than direct JSoup methods
3. **Add asserts** for optional fields to help with testing
4. **Keep parsers simple** and focused on their specific website
5. **Document any unusual workarounds** or website quirks in code comments

## Troubleshooting Common Issues
| Issue                                   | Solution                                                   |
| --------------------------------------- | ---------------------------------------------------------- |
| Network requests failing                | Check if the site requires specific headers or cookies     |
| Content not being parsed correctly      | Verify the HTML structure hasn't changed; update selectors |
| Tests passing locally but failing in CI | Ensure your parser handles network instability gracefully  |
| Getting null values                     | Add more logging to debug the parsing process              |