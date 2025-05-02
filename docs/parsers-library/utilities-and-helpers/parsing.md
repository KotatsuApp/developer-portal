# Parsing

This page documents the utilities provided by the Kotatsu Parsers library for parsing HTML and JSON data from manga websites. These utilities extend the functionality of libraries like Jsoup and Android's JSON implementation to simplify common parsing patterns and handle edge cases that frequently occur when scraping manga sites.

For information about network-related utilities, see [Network](network.md).

## 1. HTML Parsing Utilities
The Kotatsu Parsers library offers comprehensive utilities for HTML parsing built on top of Jsoup. These utilities handle common tasks such as attribute extraction, element selection, and URL manipulation.

### 1.1 Attribute Extraction
The library provides several utility functions to extract attributes from HTML elements with improved error handling compared to Jsoup's default methods.

| Function | Description | Behavior on Missing Value |
| ----------- | ------------- | -------------------- |
| `attrOrNull(attributeKey)` | Extracts attribute value | Returns `null` |
| `attrOrThrow(attributeKey)` | Extracts attribute value | Throws `ParseException` |
| `src()` | Finds image source across multiple attributes | Returns `null` |
| `requireSrc()` | Finds image source with mandatory presence | Throws `ParseException` |

These methods provide safer alternatives to Jsoup's native attr() method, which returns an empty string for missing attributes. This behavior can lead to subtle bugs when parsing manga sites where image sources might be in various attributes.

!!! example "Example usage pattern in parsers"
    ``` kotlin
    // Safely extract an optional attribute
    val altTitle = titleElement.attrOrNull("data-alt-title")

    // Extract a required attribute or fail with a clear message
    val chapterId = chapterElement.attrOrThrow("data-id")

    // Find image source across multiple common attributes
    val coverUrl = imageElement.src()
    ```

### 1.2 Element Selection
The library extends Jsoup's element selection capabilities with more robust error handling:

| Function | Description | Behavior on Missing Value |
| ----------- | ------------- | -------------------- |
| `selectFirstOrThrow(cssQuery)` | Selects first matching element | Throws `ParseException` |
| `selectOrThrow(cssQuery)` | Selects all matching elements | Throws if empty |
| `requireElementById(id)` | Finds element by ID | Throws `ParseException` |
| `selectLast(cssQuery)` | Selects last matching element | Returns `null` |
| `selectLastOrThrow(cssQuery)` | Selects last matching element | Throws `ParseException` |
| `selectFirstParent(query)` | Selects first matching parent | Returns `null` |

These utilities make the code more readable and provide better error messages when elements are not found.

### 1.3 URL Handling
The library provides utilities for handling URLs within HTML elements:

| Function | Description |
| ----------- | ------------- |
| `attrAsAbsoluteUrl(attributeKey)` | Extracts attribute as absolute URL |
| `attrAsAbsoluteUrlOrNull(attributeKey)` | Extracts attribute as absolute URL or null |
| `attrAsRelativeUrl(attributeKey)` | Extracts attribute as relative URL |
| `attrAsRelativeUrlOrNull(attributeKey)` | Extracts attribute as relative URL or null |
| `host` property | Gets the host from element's base URI |

These methods simplify the common task of extracting and normalizing URLs from HTML elements, handling edge cases like data URLs and empty values.

### 1.4 CSS Property Handling
The library includes utilities for parsing CSS properties:

| Function | Description |
| ----------- | ------------- |
| `styleValueOrNull(property)` | Extracts CSS property value from style attribute |
| `backgroundOrNull()` | Parses CSS background properties into a structured object |
| `cssUrl()` | Extracts URL from CSS `url()` function |

## 2. JSON Parsing Utilities
The library provides extension functions for Android's JSON implementation to simplify common JSON parsing tasks.

### 2.1 JSON Conversion
Functions to safely convert strings to JSON structures:

| Function | Description | On Invalid JSON |
| ----------- | ------------- | -------------------- |
| `toJSONObjectOrNull()` | Converts string to JSONObject | Returns `null` |
| `toJSONArrayOrNull()` | Converts string to JSONArray | Returns `null` |

### 2.2 JSON Collection Mapping
Extension functions for mapping JSONArrays to collections:

| Function | Description |
| ----------- | ------------- |
| `mapJSON(block)` | Maps JSONObjects to a List |
| `mapJSONNotNull(block)` | Maps non-null results to a List |
| `mapJSONToSet(mapper)` | Maps JSONObjects to a Set |
| `mapJSONNotNullToSet(mapper)` | Maps non-null results to a Set |
| `mapJSONIndexed(block)` | Maps with index information |
| `asTypedList<T>()` | Views JSONArray as a typed List |

These utilities simplify working with JSON arrays, which is common when parsing manga lists, chapter lists, and tags.

### 2.3 Type-Safe Property Extraction
Extension functions for type-safe property extraction from JSONObjects:

| Function | Description |
| ----------- | ------------- |
| `getStringOrNull(name)` | Gets string property or null |
| `getBooleanOrDefault(name, default)` | Gets boolean with fallback |
| `getIntOrDefault(name, default)` | Gets int with fallback |
| `getLongOrDefault(name, default)` | Gets long with fallback |
| `getFloatOrDefault(name, default)` | Gets float with fallback |
| `getDoubleOrDefault(name, default)` | Gets double with fallback |
| `getEnumValueOrNull(name, enumClass)` | Gets enum value or null |
| `getEnumValueOrDefault(name, default)` | Gets enum with fallback |

These functions handle type conversion and null safety in a more Kotlin-friendly way than the standard JSON API.

### 2.4 JSON Collection Utilities
Additional utilities for working with JSON collections:

| Function | Description |
| ----------- | ------------- |
| `isNullOrEmpty()` | Checks if JSONArray is null or empty |
| `toStringSet()` | Converts JSONArray to Set of strings |
| `entries<T>()` | Views JSONObject as Iterable of typed entries |

## 3. Exception Handling
The library defines several exception types specific to parsing operations:

Notable exceptions include:

* `ParseException`: Thrown when parsing fails, includes the URL for context
* `NotFoundException`: Indicates that content was not found (404)
* `ContentUnavailableException`: Indicates that content exists but is unavailable
* `TooManyRequestExceptions`: Handles rate limiting with retry information

## 4. Usage Examples
### 4.1 HTML Parsing Example
The ExHentai parser demonstrates usage of the HTML parsing utilities:

``` koltin
// Extract CSS background from an element
val preview = a.children().firstOrNull()?.extractPreview()

// Get text or null if empty
val username = doc.getElementById("userlinks")
    ?.getElementsByAttributeValueContaining("href", "showuser=")
    ?.firstOrNull()
    ?.ownText()

// Select element by ID or throw exception
val root = doc.body().requireElementById("gdt")

// Get element attribute as absolute URL
val imageUrl = doc.body().requireElementById("img").attrAsAbsoluteUrl("src")
```

### 4.2 CSS Background Parsing Example
The ExHentai parser demonstrates parsing CSS backgrounds for image previews:

``` kotlin
private fun Element.extractPreview(): String? {
    val bg = backgroundOrNull() ?: return null
    return buildString {
        append(bg.url)
        append('#')
        // rect: left,top,right,bottom
        append(bg.left)
        append(',')
        append(bg.top)
        append(',')
        append(bg.right)
        append(',')
        append(bg.bottom)
    }
}
```

### 4.3 Exception Handling Example
The ExHentai parser demonstrates handling rate limiting with `TooManyRequestExceptions`:

``` kotlin
// Check for rate limiting in response
if (text.contains("IP address has been temporarily banned", ignoreCase = true)) {
    val hours = Regex("([0-9]+) hours?").find(text)?.groupValues?.getOrNull(1)?.toLongOrNull() ?: 0
    val minutes = Regex("([0-9]+) minutes?").find(text)?.groupValues?.getOrNull(1)?.toLongOrNull() ?: 0
    val seconds = Regex("([0-9]+) seconds?").find(text)?.groupValues?.getOrNull(1)?.toLongOrNull() ?: 0
    response.closeQuietly()
    throw TooManyRequestExceptions(
        url = response.request.url.toString(),
        retryAfter = TimeUnit.HOURS.toMillis(hours)
            + TimeUnit.MINUTES.toMillis(minutes)
            + TimeUnit.SECONDS.toMillis(seconds),
    )
}
```

## Summary
The parsing utilities provided by the Kotatsu Parsers library extend common parsing libraries with manga-specific functionality and improved error handling. These utilities reduce boilerplate and help handle the inconsistencies and edge cases frequently encountered when scraping manga websites.