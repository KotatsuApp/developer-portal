# Utilities and Helpers

## Overview
The Kotatsu Parsers library includes a set of utility functions and helper classes that provide common functionality used across different parsers. These utilities simplify operations such as parsing HTTP responses, manipulating URLs, working with JSON data, and handling coroutines. This page documents these utilities, which serve as the foundation for the parser implementations.

For information about parser implementations that use these utilities, see Base Parser Implementations and Site-Specific Parsers. For details about network-specific utilities, see Network Utilities.

## Parsing Utilities
The parsing utilities provide functions for parsing HTTP responses and manipulating URLs, which are essential operations for manga parsers that need to extract data from web pages.

### HTTP Response Parsing
The Kotatsu Parsers library includes extension functions for `okhttp3.Response` that make it easy to parse response bodies into various formats:

These functions allow parsers to easily extract and process data from HTTP responses:

* `parseHtml()`: Parses the response body as an HTML document using Jsoup
* `parseJson()`: Parses the response body as a JSON object
* `parseJsonArray()`: Parses the response body as a JSON array
* `parseRaw()`: Returns the response body as a string
* `parseBytes()`: Returns the response body as a byte array
* `requireBody()`: Gets the response body or throws an exception if it's null

### URL Manipulation
The library provides utilities for working with URLs, which is important for constructing links to manga pages and chapters.

These functions handle the common URL operations needed by parsers:

* `toRelativeUrl(domain)`: Converts an absolute URL to a relative URL if it belongs to the specified domain
* `toAbsoluteUrl(domain)`: Converts a relative URL to an absolute URL using the specified domain
* `concatUrl(host, path)`: Concatenates a host and path, handling slash characters correctly

### Date Parsing
The library includes a utility for safely parsing dates:

* `DateFormat.tryParse(str)`: Attempts to parse a date string, returning 0L if parsing fails

## JSON Utilities
The JSON utilities provide extension functions for working with `JSONObject` and `JSONArray`, making it easier to extract and transform JSON data.

### JSON Object Extensions
These extensions provide safer ways to extract values from JSON objects:

* `getStringOrNull(name)`: Gets a string value or null if it doesn't exist
* `getBooleanOrDefault(name, defaultValue)`: Gets a boolean value or returns the default
* `getLongOrDefault(name, defaultValue)`: Gets a long value or returns the default
* `getIntOrDefault(name, defaultValue)`: Gets an int value or returns the default
* `getDoubleOrDefault(name, defaultValue)`: Gets a double value or returns the default
* `getFloatOrDefault(name, defaultValue)`: Gets a float value or returns the default
* `getEnumValueOrNull(name, enumClass)`: Gets an enum value or null
* `getEnumValueOrDefault(name, defaultValue)`: Gets an enum value or returns the default
* `entries(typeClass)`: Gets entries of a specific type from the JSON object

### JSON Array Extensions
These extensions make it easier to transform JSON arrays into Kotlin collections:

* `mapJSON(block)`: Maps each JSONObject in the array to a value
* `mapJSONTo(destination, block)`: Maps each JSONObject to a value and adds it to the destination collection
* `mapJSONNotNull(block)`: Maps each JSONObject to a non-null value
* `mapJSONNotNullTo(destination, block)`: Maps each JSONObject to a non-null value and adds it to the destination collection
* `mapJSONToSet(mapper)`: Maps each JSONObject to a value and returns a Set
* `mapJSONNotNullToSet(mapper)`: Maps each JSONObject to a non-null value and returns a Set
* `mapJSONIndexed(block)`: Maps each JSONObject with its index
* `isNullOrEmpty()`: Checks if the array is null or empty
* `asTypedList(typeClass)`: Wraps the array as a typed list
* `toStringSet()`: Converts the array to a set of strings

### JSON String Parsing
The library includes utilities for safely parsing JSON strings:

* `String.toJSONObjectOrNull()`: Attempts to parse the string as a JSON object
* `String.toJSONArrayOrNull()`: Attempts to parse the string as a JSON array

## Coroutine Utilities
The coroutine utilities provide functions for working with Kotlin coroutines, particularly for handling jobs and deferred values.

These utilities help with common coroutine operations in the parsers:

* `Iterable<Job>.cancelAll(cause)`: Cancels all jobs in the collection
* `Iterable<Deferred<T>>.awaitFirst()`: Awaits the first deferred value to complete and cancels the rest
* `Collection<Deferred<T>>.awaitFirst(condition)`: Awaits the first deferred value that satisfies a condition and cancels the rest