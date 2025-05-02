# Build System

This document describes the build system used by the Kotatsu Parsers library. It covers the Gradle configuration, project structure, build processes, and code generation mechanisms used to compile and package the parser implementations.

## Overview
The Kotatsu Parsers project uses Gradle as its build system with Kotlin DSL. The build configuration supports:

* Kotlin compilation targeting JVM
* Kotlin Symbol Processing (KSP) for code generation
* Maven publishing for distribution
* Custom tasks for generating test reports

## Project Modules
### Root Module (kotatsu-parsers)
The main module contains all the parser implementations and core interfaces that define the parsing framework.

### KSP Module (kotatsu-parsers-ksp)
A separate module that contains the Kotlin Symbol Processor for generating code from annotations in the main module.

### BuildSrc Module
Contains custom Gradle tasks used during the build process, particularly for test report generation.

## Gradle Configuration
### Main Build Configuration
The main build file defines the project as a Kotlin/Java library with the following key configurations:

* Kotlin version: 2.0.20
* Java compatibility: JVM 8
* Maven publishing setup
* Required dependencies
* Compiler configurations

#### Key Dependencies
| Dependency | Version | Purpose |
| -------------- | ------------- | ---------- |
| kotlinx-coroutines-core | 1.10.2 | Asynchronous programming |
| okhttp | 4.12.0 | HTTP client |
| jsoup | 1.19.1 | HTML parsing |
| json | 20240303 | JSON parsing |
| junit-jupiter | 5.10.1 | Testing framework |
| quickjs | 1.1.0 | JavaScript execution for tests |

### Kotlin Configuration
The project uses explicit API mode for better encapsulation and documentation:

``` groovy
kotlin {
    jvmToolchain(8)
    explicitApi = 'warning'
    sourceSets {
        main.kotlin.srcDirs += 'build/generated/ksp/main/kotlin'
    }
}
```

Compiler options include opt-ins for experimental features:

``` groovy
compileKotlin {
    kotlinOptions {
        freeCompilerArgs += [
            '-opt-in=kotlin.RequiresOptIn',
            '-opt-in=kotlin.contracts.ExperimentalContracts',
            '-opt-in=kotlinx.coroutines.ExperimentalCoroutinesApi',
            '-opt-in=org.koitharu.kotatsu.parsers.InternalParsersApi',
        ]
    }
}
```

## Code Generation System
### Annotation Processing
The project uses Kotlin Symbol Processing (KSP) to generate code from annotations. The main annotation processed is `@MangaSourceParser`, which is used to define parser implementations.

The KSP processor generates:

1. **MangaParserFactory.kt** - A factory class that creates parser instances based on `MangaParserSource` enum values
2. **MangaSource.kt** - An enum of all available manga sources with metadata

### Parser Annotation System
Parsers are defined using the `@MangaSourceParser` annotation which includes:

* Name (enum value)
* Title (user-friendly name)
* Locale (language code)
* Content type
!!! example
    ``` kotlin
    @MangaSourceParser("MANGARBIC", "MangaArabic", "ar")
    internal class Mangarbic(context: MangaLoaderContext) :
        MadaraParser(context, MangaParserSource.MANGARBIC, "lekmanga.net") {
        override val postReq = true
        override val datePattern = "yyyy/MM/dd"
    }
    ```

Broken parsers are marked with the `@Broken` annotation:

``` kotlin
@Broken("Website is down or domain has been changed")
@MangaSourceParser("MANGARBIC", "MangaArabic", "ar")
```

## Custom Build Tasks
### Test Report Generation
The project includes a custom Gradle task for generating HTML test reports from JUnit XML results.

The `ReportGenerateTask` parses XML test results and generates human-readable HTML reports with test statistics.

## CI/CD Integration
The project includes GitHub Actions workflows for continuous integration.

The `test-parsers.yml` workflow runs when changes are made to parser implementations, ensuring that code changes don't break existing functionality.

### Publishing Configuration
The project is configured to publish Maven artifacts using the `maven-publish` plugin:

``` kotlin
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

This configuration allows the library to be published to Maven repositories and consumed by other projects.

### Building the Project
To build the project, you can use the following commands:

* `./gradlew build` - Compiles the code and runs tests
* `./gradlew assemble` - Compiles the code without running tests
* `./gradlew generateTestsReport` - Generates HTML test reports
* `./gradlew publishToMavenLocal` - Publishes the library to the local Maven repository
