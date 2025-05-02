# Data Models

This page documents the core data models in the Kotatsu Parsers library. These models represent the essential entities that make up the manga ecosystem - manga titles, chapters, pages, tags, and other supporting classes. The data models provide a standardized structure for parsers to populate with data from various manga sources.

For information about how search and filtering works with these models, see Search and Filter System. For details on the parent MangaParser interface that manipulates these models, see MangaParser Interface.

## Core Data Models
### Manga
The central data model that represents a manga title with all its metadata and references.

#### Key Properties

| Property | Type | Description |
| --------- | --------------- | ---------------- |
| `id` | Long | Unique identifier for the manga |
| `title` | String | Primary title of the manga |
| `altTitles` | Set | Alternative titles (e.g., in different languages) |
| `url` | String | Relative URL (without domain) used by parsers |
| `publicUrl` | String | Absolute URL for browser access |
| `rating` | Float | Normalized rating (0-1) or `RATING_UNKNOWN` (-1f) |
| `contentRating` | ContentRating? | Content appropriateness level |
| `coverUrl` | String? | URL to the manga's cover image |
| `tags` | Set | Associated genres/tags |
| `state` | MangaState? | Publication state (ongoing, finished, etc.) |
| `authors` | Set | Set of author names |
| `chapters` | List? | List of available chapters |
| `source` | MangaSource | Source/website the manga is from | 

#### Utility Methods
* `hasRating`: Checks if manga has a valid rating (between 0 and 1)
* `getChapters(branch)`: Filters chapters by specific branch
* `findChapterById(id)`: Retrieves a specific chapter by ID
* `getBranches()`: Returns map of branches with their chapter counts

### MangaChapter
Represents a single chapter within a manga.

#### Key Properties

| Property | Type | Description |
| --------- | --------------- | ---------------- |
| `id` | Long | Unique identifier for the chapter |
| `title` | String? | User-readable name (optional) |
| `number` | Float | Chapter number (1+ or 0 if unknown) |
| `volume` | Int | Volume number (1+ or 0 if unknown) |
| `url` | String | Relative URL/identifier for parser use |
| `scanlator` | String? | Translation group name (optional) |
| `uploadDate` | Long | Timestamp of chapter upload |
| `branch` | String? | Group identifier for overlapping chapters (e.g., translations) |
| `source` | MangaSource | Source of the chapter |

#### Methods
* `numberString()`: Returns formatted chapter number string or null
* `volumeString()`: Returns volume number as string or null

### MangaPage
Represents a single page within a manga chapter.

#### Properties

| Property | Type | Description |
| --------- | --------------- | ---------------- |
| `id` | Long | Unique identifier for the page |
| `url` | String | Relative URL to page image or HTML page |
| `preview` | String? | Optional URL to thumbnail/preview |
| `source` | MangaSource | Source of the page |

### MangaTag
Represents a genre, category, or other metadata associated with manga.

| Property | Type | Description |
| --------- | --------------- | ---------------- |
| `title` | String | User-readable title (in Title Case) |
| `key` | String | Identifier unique within the source |
| `source` | MangaSource | Source of the tag |

## Enumeration Types
### MangaState
Represents the publication status of a manga.

| Value | Description |
| --------- | ---------------- |
| `ONGOING` | Currently publishing |
| `FINISHED` | Completed publication |
| `ABANDONED` | Discontinued or canceled |
| `PAUSED` | Temporarily on hiatus |
| `UPCOMING` | Announced but not yet published |

### ContentRating
Indicates the content appropriateness level.

| Value | Description |
| --------- | ---------------- |
| `SAFE` | General audience content |
| `SUGGESTIVE` | Content with mature themes or suggestive material |
| `ADULT` | Explicit or adult-only content |

### ContentType
Classifies the format of the content.

| Value | Description |
| --------- | ---------------- |
| `MANGA` | Japanese comics |
| `MANHWA` | Korean comics |
| `MANHUA` | Chinese comics |
| `COMIC` | Western comics |
| `NOVEL` | Text-based content |
| `OTHER` | Other formats |

### Demographic
Indicates the target audience demographic.

| Value | Description |
| --------- | ---------------- |
| `SHOUNEN` | Young male readers |
| `SHOUJO` | Young female readers |
| `SEINEN` | Adult male readers |
| `JOSEI` | Adult female readers |
| `NONE` | No specific demographic |

## Filtering and Search Models
### MangaListFilter (Legacy)
Model for filtering manga lists based on various criteria.

!!! note 
    This is being replaced by the newer search system (see [Search and Filter System](search-and-filter-system.md)).

#### Key Properties
| Property | Type | Description |
| --------- | --------------- | ---------------- |
| `query` | String? | Search text |
| `tags` | Set | Tags to include |
| `tagsExclude` | Set | Tags to exclude |
| `locale` | Locale? | Language filter |
| `originalLocale` | Locale? | Original language filter |
| `states` | Set | Translation group name (optional) |
| `contentRating` | Set | Timestamp of chapter upload |
| `types` | Set | Group identifier for overlapping chapters (e.g., translations) |
| `demographics` | Set | Source of the chapter |
| `year`, `yearFrom`, `yearTo` | Int | Source of the chapter |
| `author` | String? | Source of the chapter |

## Additional Models
### Favicons
Represents source website favicons with a collection of different sizes and formats.

## Constants
Several constants are defined for use in the data models:

| Constant | Value | Description |
| --------- | --------------- | ---------------- |
| `RATING_UNKNOWN` | -1f | Used for manga without rating information |
| `YEAR_UNKNOWN` | 0 | Used when publication year is unknown |
| `YEAR_MIN` | 1900 | Minimum valid publication year |
| `YEAR_MAX` | 2099 | Maximum valid publication year |

## ID Generation System
The library uses a consistent hashing algorithm to generate unique IDs for manga, chapters, and pages. This ensures that entities have unique IDs across different sources, even if they have the same URL or internal identifier.

The ID generation is implemented in `MangaParserEnv.kt` with the `generateUid` functions.

## Real-World Usage Examples
To illustrate how these models are used in practice, consider these examples from actual parsers:

### MangaDex Parser Example
The MangaDex parser constructs a Manga object from JSON data:

``` kotlin
// From MangaDexParser.kt
private fun JSONObject.fetchManga(chapters: List<MangaChapter>?): Manga {
    val id = getString("id")
    val attrs = getJSONObject("attributes")
    // ... process data ...
    
    return Manga(
        id = generateUid(id),
        title = requireNotNull(attrs.getJSONObject("title").selectByLocale()),
        altTitles = setOfNotNull(...),
        url = id,
        publicUrl = "https://$domain/title/$id",
        rating = RATING_UNKNOWN,
        contentRating = when (attrs.getStringOrNull("contentRating")) {
            "pornographic" -> ContentRating.ADULT
            "erotica", "suggestive" -> ContentRating.SUGGESTIVE
            "safe" -> ContentRating.SAFE
            else -> null
        },
        coverUrl = cover?.plus(".256.jpg"),
        largeCoverUrl = cover,
        description = attrs.optJSONObject("description")?.selectByLocale(),
        tags = attrs.getJSONArray("tags").mapJSONToSet { tag ->
            MangaTag(
                title = tag.getJSONObject("attributes")
                    .getJSONObject("name")
                    .firstStringValue()
                    .toTitleCase(),
                key = tag.getString("id"),
                source = source,
            )
        },
        state = when (attrs.getStringOrNull("status")) {
            "ongoing" -> MangaState.ONGOING
            "completed" -> MangaState.FINISHED
            "hiatus" -> MangaState.PAUSED
            "cancelled" -> MangaState.ABANDONED
            else -> null
        },
        authors = authors,
        chapters = chapters,
        source = source,
    )
}
```

### Webtoons Parser Example
The Webtoons parser shows how chapters are constructed:

``` kotlin
// From WebtoonsParser.kt
private suspend fun fetchEpisodes(titleNo: Long): List<MangaChapter> = coroutineScope {
    // ... API calls and processing ...
    
    episodes.mapChapters { i, jo ->
        MangaChapter(
            id = generateUid("$titleNo-$i"),
            title = jo.getStringOrNull("episodeTitle"),
            number = jo.getInt("episodeSeq").toFloat(),
            volume = 0,
            url = "$titleNo-${jo.get("episodeNo")}",
            uploadDate = jo.getLong("registerYmdt"),
            branch = null,
            scanlator = null,
            source = source,
        )
    }.sortedBy(MangaChapter::number)
}
```

## Best Practices
When working with these data models:

1. **Always check for nulls** - Many properties are optional and may be null
2. **Handle branches properly** - Chapters may be grouped into branches (languages/scanlation groups)
3. **Respect content ratings** - Honor content rating when displaying manga
4. **Generate proper IDs** - Use the provided `generateUid()` functions for consistent ID generation
5. **Use relative URLs for internal use** - The `url` field should contain paths without domains
6. **Use absolute URLs for public access** - The `publicUrl` field should be fully qualified
