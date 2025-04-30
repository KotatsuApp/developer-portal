# Installation and Usage

This document covers how to integrate the Kotatsu Parsers library into your project and the basic patterns for using it. For information about the core architecture and available parsers, see [Core Architecture](core-architecture/index.md).

## Library Overview
Kotatsu Parsers is a JVM library that provides a collection of manga parsers for accessing web-based manga content. It supports both standard JVM applications and Android projects.

## Installation
### Adding the Repository
First, add JitPack repository to your root `build.gradle` file:

``` groovy
allprojects {
    repositories {
        ...
        maven { url 'https://jitpack.io' }
    }
}
```

### Adding the Dependency
#### For Java/Kotlin Projects

``` groovy
dependencies {
    implementation("com.github.KotatsuApp:kotatsu-parsers:$parsers_version")
}
```

#### For Android Projects
For Android projects, exclude the `org.json:json` module to avoid conflicts with Android's built-in JSON library:

``` groovy
dependencies {
    implementation("com.github.KotatsuApp:kotatsu-parsers:$parsers_version") {
        exclude group: 'org.json', module: 'json'
    }
}
```

!!! note
    When used in Android projects, core library desugaring with the NIO specification should be enabled to support Java 8+ features.

### Dependencies
The library includes the following key dependencies:

| Dependency              | Version  | Purpose                  |
| ----------------------- | -------- | ------------------------ |
| kotlinx-coroutines-core | 1.10.2   | Asynchronous programming |
| okhttp3                 | 4.12.0   | HTTP client              |
| jsoup                   | 1.19.1   | HTML parsing             |
| org.json                | 20240303 | JSON parsing             |

## Basic Usage
### Implementing MangaLoaderContext
Before using any parsers, you need to implement the `MangaLoaderContext` interface, which provides the necessary context for parsers to function.

The implementation differs between Android and non-Android projects. Examples of implementations can be found in:

* [Kotatsu](https://github.com/KotatsuApp/Kotatsu/blob/devel/app/src/main/kotlin/org/koitharu/kotatsu/core/parser/MangaLoaderContextImpl.kt)
* [kotatsu-dl (non-android)](https://github.com/KotatsuApp/kotatsu-dl/blob/master/src/jvmMain/kotlin/org/koitharu/kotatsu_dl/logic/MangaLoaderContextImpl.kt)

### Creating a Parser Instance
Once you have implemented `MangaLoaderContext`, you can create parser instances:

``` kotlin
val parser = mangaLoaderContext.newParserInstance(MangaParserSource.MANGADEX)
```

Where `MangaParserSource` is an enumeration of all available manga sources.

!!! warning
    `MangaParserSource.DUMMY` parser cannot be instantiated.

### Key Operations
The main operations available through parsers are:

1. **Getting manga list**: `getList(query, offset, order, filter)` - Retrieves a list of manga with pagination
2. **Getting manga details**: `getDetails(manga)` - Fetches detailed information about a manga, including chapters
3. **Getting chapter pages**: `getPages(chapter)` - Retrieves a list of pages for a specific chapter
4. **Getting page images**: `getPageUrl(page)` - Gets the direct URL to an image

## Projects Using the Library
Several projects are already using the Kotatsu Parsers library:

* [Kotatsu](https://github.com/KotatsuApp/Kotatsu) - Android manga reader application
* [kotatsu-dl](https://github.com/KotatsuApp/kotatsu-dl) - Command-line manga downloader
* [Shirizu](https://github.com/ztimms73/shirizu) - Work in progress manga reader
* [OtakuWorld](https://github.com/jakepurple13/OtakuWorld) - Manga and anime app

These projects can serve as examples of how to integrate and use the library in your own applications.