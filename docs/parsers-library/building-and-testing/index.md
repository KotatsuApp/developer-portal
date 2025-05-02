# Building and Testing

This document provides instructions for building and testing the Kotatsu Parsers library. It explains how to set up your development environment, build the project, and run the automated test suite to validate parser functionality.

For information about adding new parsers, see [Contributing](../contributing/index.md) and [Adding a New Parser](../contributing/adding-a-new-parser.md).

## Setting Up the Development Environment
Before building the project, ensure you have the following prerequisites installed:

* JDK 8 or newer
* Gradle (or use the included Gradle wrapper)
The project uses Kotlin 2.0.20 and the Kotlin Symbol Processing (KSP) plugin for annotation processing.

## Building the Project
To build the project, run the Gradle build command from the project root directory:

``` bash
./gradlew build
```

For just compiling without running tests:

``` bash
./gradlew assemble
```

The build process involves:

1. Compiling the Kotlin code
2. Processing annotations with KSP
3. Running tests
4. Generating the library JAR file

The build will generate:

* Library JAR in `build/libs/`
* KSP outputs in `build/generated/ksp/`
* Test reports in `build/reports/tests/`

## Testing Framework
Kotatsu Parsers uses JUnit 5 (Jupiter) for testing. The main test class `MangaParserTest` provides a comprehensive test suite that validates all manga parsers against a common specification.

### Test Configuration
Tests are configured in the `build.gradle` file:

``` kotlin
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.10.1'
    testImplementation 'org.junit.jupiter:junit-jupiter-engine:5.10.1'
    testImplementation 'org.junit.jupiter:junit-jupiter-params:5.10.1'
    testImplementation 'org.jetbrains.kotlinx:kotlinx-coroutines-test:1.10.2'
    testImplementation 'io.webfolder:quickjs:1.1.0'
}

test {
    useJUnitPlatform()
}
```

### Running Tests
To run tests for all parsers:

``` bash
./gradlew test
```

To run tests for a specific parser, you can use the test filter:

``` bash
./gradlew test --tests "org.koitharu.kotatsu.parsers.MangaParserTest.list[MANGADEX]"
```

### Test Report Generation
The project includes a custom test report generator that creates HTML reports from test results:

``` bash
./gradlew generateTestsReport
```

This task processes the XML test results and generates a more readable HTML report showing test status for each parser.

### What the Tests Validate
The test suite validates each parser's implementation across multiple aspects:

| Test Method | Description |
| ------------ | ----------- |
| `list` | Verifies that the parser can retrieve a list of manga |
| `pagination` | Checks that pagination works correctly |
| `searchByTitleName` | Tests search functionality by manga title |
| `tags` | Verifies tag filtering |
| `tagsMultiple` | Tests filtering by multiple tags |
| `locale` | Validates locale-based filtering |
| `details` | Ensures manga details are correctly parsed |
| `pages` | Verifies chapter page retrieval |
| `favicon` | Checks for correct favicon parsing |
| `domain` | Validates domain configuration |
| `link` | Tests link resolution |
| `authorization` | Validates authorization if applicable (disabled by default) |

Each test method is annotated with `@ParameterizedTest` and `@MangaSources`, which runs the test against all supported manga sources.

## Continuous Integration
The project uses GitHub Actions for continuous integration. The workflow defined in `.github/workflows/test-parsers.yml` runs tests automatically when changes are made to parser implementations.

The CI workflow:

1. Checks out the code
2. Sets up Java 11
3. Builds the project
4. Runs tests

This ensures that all changes to parsers are validated before merging.

## Common Testing Issues
When implementing or modifying parsers, be aware of these common issues:

1. **Timeout Errors**: Tests have a timeout of 2 minutes. If a parser takes too long to respond, the test will fail.
2. **Domain Validation**: Tests verify that the configured domain matches the actual domain used in HTTP requests.
3. **Response Format**: Tests verify that manga, chapters, and pages have the expected structure and fields.
4. **Distinct Values**: Chapter IDs, Manga IDs, and other identifiers must be unique within their respective collections.
5. **Source Consistency**: All items returned by a parser must have the correct source identifier.

The test suite includes comprehensive validation to catch these and other issues.

## Publishing
The project includes Maven publishing configuration to publish the library as a Maven artifact:

``` groovy
afterEvaluate {
    publishing {
        publications {
            mavenJava(MavenPublication) {
                from components.java
            }
        }
    }
}
```

To publish the library:

``` bash
./gradlew publish
```