# Other

This page documents the various utility classes and extension functions provided by the Kotatsu Parsers library that don't fit neatly into other categories. These utilities help with common operations like collection manipulation, HTTP handling, number formatting, and more. For network-specific utilities, see [Network](network.md), and for parsing-specific utilities, see [Parsing](parsing.md).

## Collection Utilities
The Collection utilities provide extension functions for more efficient collection manipulation. These functions are defined in the `org.koitharu.kotatsu.parsers.util` package.

### Key Collection Extensions
The library provides numerous helpful collection extensions:

| Function | Description |
| -------------- | ----------- |
| `Collection<*>?.sizeOrZero()` | Returns the size of the collection or 0 if it's null |
| `MutableCollection<T>.replaceWith(subject: Iterable<T>)` | Clears the collection and adds all elements from the subject |
| `List<T>.medianOrNull()` | Returns the median element of a list or null if empty |
| `Collection<T>.mapToSet(transform)` | Maps a collection to a set with a transform function |
| `MutableList<T>.move(sourceIndex, targetIndex)` | Moves an element from one position to another |
| `Iterator<T>.nextOrNull()` | Returns the next element or null if there are no more elements |
| `Collection<T>.associateGrouping(transform)` | Groups elements by keys with values in sets |

### Example Usage

``` kotlin
// Getting size safely
val count = someNullableCollection.sizeOrZero()

// Finding the median element
val median = listOf(1, 2, 3, 4, 5).medianOrNull() // Returns 3

// Moving elements in a list
val list = mutableListOf("a", "b", "c", "d")
list.move(0, 2) // Results in ["b", "c", "a", "d"]

// Incrementing a counter in a map
val countMap = mutableMapOf<String, Int>()
countMap.incrementAndGet("key") // Returns 1 and updates the map
```

## HTTP and Network Utilities
The library provides utilities for working with OkHttp, including extension functions for handling HTTP requests and cookies.

### OkHttp Extensions
The library extends OkHttp's functionality with suspension functions and helpful properties:

| Extension | Description |
| -------------- | ----------- |
| `Call.await()` | Suspending version of `Call.execute()` for use in coroutines |
| `Response.mimeType` | Property to get the MIME type from a response |
| `HttpUrl.isHttpOrHttps` | Property to check if a URL uses HTTP or HTTPS |
| `Headers.Builder.mergeWith()` | Merges two sets of headers |
| `Response.copy()` | Creates a copy of a response with its body |
| `Response.map()` | Maps a response body to a new one |

### Cookie Management
Extensions for OkHttp's CookieJar to simplify cookie operations:

| Function | Description |
| -------------- | ----------- |
| `CookieJar.insertCookies(domain, vararg cookies)` | Inserts cookies for a domain |
| `CookieJar.insertCookie(domain, cookie)` | Inserts a single cookie for a domain |
| `CookieJar.getCookies(domain)` | Gets all cookies for a domain |
| `CookieJar.copyCookies(oldDomain, newDomain, names)` | Copies cookies from one domain to another |

## Number Formatting and Manipulation
The library provides utilities for number formatting and manipulation.

### Number Formatting
The library includes functions for formatting numbers in various ways:

| Function | Description |
| -------------- | ----------- |
| `Number.format(decimals, decPoint, thousandsSep)` | Formats a number with custom decimal and thousand separators |
| `Number.formatSimple()` | Formats a number by removing trailing ".0" |
| `Float.toIntUp()` | Converts a float to an integer, rounding up if necessary |
| `Int.upBy(step)` | Rounds up an integer to the nearest multiple of step |
| `Int/Long.ifZero(defaultValue)` | Returns the number or a default value if zero |

!!! example
    ``` kotlin
    val number = 1234.56
    number.format(decimals = 2) // "1 234.56"
    number.formatSimple() // "1234.56"

    val intValue = 5.0f.toIntUp() // 5
    val roundedUp = 17.upBy(5) // 20

    val value = 0.ifZero { getDefaultValue() } // Calls getDefaultValue()
    ```

## Suspension Utilities
The library provides utilities for working with suspending functions.

### SuspendLazy
The `SuspendLazy` interface provides lazy initialization for suspending functions, similar to Kotlin's standard `Lazy`, but for coroutines:

``` kotlin
// Creating a suspend lazy property
val lazyData = suspendLazy {
    // Do some suspending work
    fetchDataFromNetwork()
}

// Using the lazy property
val data = lazyData.get() // Initializes only on first call

// Checking if initialized
if (lazyData.isInitialized) {
    // Use the data
}

// Getting the value or null if an error occurs
val safeData = lazyData.getOrNull()

// Getting the value or a default if an error occurs
val dataWithDefault = lazyData.getOrDefault("default")
```

The library provides two implementations:

* `SuspendLazyImpl`: Standard implementation that keeps a strong reference to the initialized value
* `SoftSuspendLazyImpl`: Uses a soft reference, allowing the value to be garbage collected if needed

## Manga-Specific Utilities
The library includes utilities specific to manga operations.

### RelatedMangaFinder
The `RelatedMangaFinder` class helps find manga related to a given manga by searching for titles with similar keywords:

``` kotlin
// Create a RelatedMangaFinder with a collection of parsers
val finder = RelatedMangaFinder(parsers)

// Find manga related to the seed manga
val relatedManga = finder(seedManga)
```

!!! info "How it works"
    1. Extracts keywords from the title and alternative titles of the seed manga
    2. Searches for each keyword using the available parsers
    3. Filters results to remove the original manga and keep only those containing the searched keyword
    4. Returns the smallest non-empty result set to provide the most relevant matches

### MangaParsers Utilities
Additional utility functions for manga parsers:

``` kotlin
// Check if a filter is null or empty
if (filter.isNullOrEmpty()) {
    // Handle empty filter case
}

// Find a chapter by ID
val chapter = chapters.findById(chapterId)
```

## Testing Utilities
The library provides utility functions specifically for testing parsers.

These test utilities help verify parser implementations by providing functions to:

* Check if collections contain distinct elements
* Validate URL formats
* Create test manga instances
* Extend destructuring declarations for lists

!!! example
    ``` kotlin
    // Check if manga titles are unique
    val titles = mangaList.map { it.title }
    assert(titles.isDistinct())

    // Check if a URL is absolute
    assert("https://example.com/manga".isUrlAbsolute())

    // Create a test manga instance
    val testManga = mangaOf(MangaParserSource.MANGADEX, "https://mangadex.org/title/123")
    ```