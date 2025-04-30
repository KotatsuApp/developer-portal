# Network

This document covers the network-related utilities provided by the Kotatsu Parsers library. These utilities facilitate HTTP request handling, response parsing, URL manipulation, and handling of special cases like CloudFlare protection. For information about HTML and JSON parsing utilities specifically, see Parsing Utilities.

## Response Parsing
The library provides extension functions for `okhttp3.Response` to parse response bodies into various formats:

| Function           | Return Type     | Description                                                          |
| ------------------ | --------------- | -------------------------------------------------------------------- |
| `parseHtml()`      | `Document`      | Parses the response body as an HTML document using Jsoup             |
| `parseJson()`      | `JSONObject`    | Parses the response body as a JSONObject                             |
| `parseJsonArray()` | `JSONArray`     | Parses the response body as a JSONArray                              |
| `parseRaw()`	     | `String`        | Parses the response body as a raw string                             |
| `parseBytes()`     | `ByteArray`     | Parses the response body as a byte array                             |
| `requireBody()`    | `ResponseBody`  | Requires that the response has a body, throwing an exception if null |

## URL Handling
The URL utilities help in converting between relative and absolute URLs, which is necessary for processing manga page links:

| Function                       | Return Type                                         | Example                                                                      |
| ------------------------------ | --------------------------------------------------- | ---------------------------------------------------------------------------- |
| `String.toRelativeUrl(domain)` | Converts URL to relative if on the specified domain | `"https://example.com/manga".toRelativeUrl("example.com")` → `"/manga"`      |
| `String.toAbsoluteUrl(domain)` | Converts to absolute URL with the specified domain  | `"/manga".toAbsoluteUrl("example.com")` → `"https://example.com/manga"`      |
| `concatUrl(host, path)`        | Concatenates host and path, handling slashes        | `concatUrl("https://example.com", "/manga")` → `"https://example.com/manga"` |

## CloudFlare Protection Handling
Many manga websites employ CloudFlare protection that can block automated requests. The library provides tools to detect and handle such protection:

### CloudFlare Detection Constants
| Constant                  | Value | Description                           |
| ------------------------- | ----- | ------------------------------------- |
| `PROTECTION_NOT_DETECTED` | 0     | No CloudFlare protection detected     |
| `PROTECTION_CAPTCHA`      | 1     | CloudFlare requires solving a captcha |
| `PROTECTION_BLOCKED`      | 2     | CloudFlare has blocked access         |

### CloudFlare Helper Methods
| Method                                 | Description                                                         |
| -------------------------------------- | ------------------------------------------------------------------- |
| `checkResponseForProtection(response)` | Examines a response to determine if CloudFlare protection is active |
| `getClearanceCookie(cookieJar, url)`   | Retrieves the CloudFlare clearance cookie (`cf_clearance`)          |
| `isCloudFlareCookie(name)`             | Determines if a cookie is CloudFlare-related by name pattern        |

## Favicon Parsing
The `FaviconParser` class extracts website favicon information:

### Favicon Parsing Process

1. Fetch the website's HTML using the `WebClient`
2. Look for a web manifest link in the HTML
3. If found, parse the manifest to extract icon information
4. Parse favicon links in the HTML (`<link rel="icon">`, etc.)
5. If no icons are found, create a fallback favicon at `/favicon.ico`
6. Return a `Favicons` object containing all found icons

## Coroutine Utilities for Network Operations
The library includes coroutine utilities to help with concurrent network operations:

| Function                                        | Description                                                  |
| ----------------------------------------------- | ------------------------------------------------------------ |
| `Iterable<Job>.cancelAll(cause)`                | Cancels all jobs in the iterable with an optional cause      |
| `Iterable<Deferred<T>>.awaitFirst()`            | Awaits the first completed deferred value and cancels others |
| `Collection<Deferred<T>>.awaitFirst(condition)` | Awaits the first deferred value that satisfies the condition |

These utilities are particularly useful for:

* Making requests to multiple mirrors of a manga site and using the first successful response
* Implementing timeout behavior by canceling jobs
* Racing multiple potential data sources