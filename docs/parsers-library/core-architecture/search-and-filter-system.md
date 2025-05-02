# Search and Filter System

The Kotatsu parsers library provides a robust search and filter system that allows users to find manga across different sources using various criteria. This document details the architecture, components, and usage of this system, which is central to the manga discovery experience.

## Overview
The Search and Filter System enables querying manga sources with type-safe criteria such as title, tags, content rating, and publication status. It provides a unified interface that parsers implement to support consistent search functionality across diverse manga sources.

## Core Components
### MangaSearchQuery
The primary class representing a search request. It contains:

* A list of filtering criteria
* An offset for pagination
* A sort order preference

### QueryCriteria Hierarchy

``` mermaid
classDiagram
  QueryCriteria <|-- Match
  QueryCriteria <|-- Range
  QueryCriteria <|-- Exclude
  QueryCriteria <|-- Include
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
  class QueryCriteria{
    <<interface>>
    +field: SearchableField
  }
```

Each criterion type serves a specific filtering purpose:

* **Include**: Results must match at least one of the provided values
* **Exclude**: Results must not match any of the provided values
* **Range**: Results must fall within the specified range (e.g., publication years)
* **Match**: Results must match the exact value (e.g., title search)

### SearchableField
An enum defining all possible fields that can be searched

### MangaSearchQueryCapabilities
Defines what search capabilities are available for a specific parser

* `capabilities`: A set of SearchCapability objects defining what each parser supports
* `validate()`: Verifies that a query conforms to the parser's capabilities

## Implementing Search in a Parser
Parsers must implement search functionality by:

1. Defining their search capabilities
2. Implementing the `getList(query: MangaSearchQuery)` method
3. Converting query criteria to source-specific URL parameters

### Example: Processing Search Queries
Parsers typically build HTTP requests based on the search criteria:

1. Start with a base URL
2. Process each criterion based on its type
3. Add sort order and pagination
4. Execute the request and parse results

## Testing Search Functionality
The `MangaParserTest` class shows how the search system is tested:

* **Title Search Test**: Verifies that searching by title returns expected results
* **Tag Filter Test**: Checks filtering by a specific tag
* **Multiple Tags Test**: Tests filtering with multiple tags
* **Locale Filter Test**: Validates filtering by language

## Building Search Queries
To create a search query, use the `MangaSearchQuery.Builder`:

``` kotlin
val query = MangaSearchQuery.Builder()
    .offset(0)
    .order(SortOrder.UPDATED)
    .criterion(Match(TITLE_NAME, "Manga Title"))
    .criterion(Include(TAG, setOf(tag1, tag2)))
    .criterion(Exclude(TAG, setOf(tag3)))
    .criterion(Include(STATE, setOf(MangaState.ONGOING)))
    .build()
```

## Error Handling
The system includes validation that throws specific exceptions:

* **Unsupported Fields**: Throws when a query uses fields not supported by the parser
* **Unsupported Criteria Types**: Throws when using unsupported criterion types for a field
* **Multiple Values**: Throws when providing multiple values for a field that doesn't support it
* **Exclusive Fields**: Throws when combining an exclusive field with other criteria

## Related Systems
For information about how different parsers implement their specific filtering capabilities, see Base Parser Implementations.

For details on the data models used in search queries and results, see [Data Models](data-models.md).