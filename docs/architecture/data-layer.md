---
title: Data Layer
description: This page documents the data persistence system of Kotatsu.
---

# Data Layer

This page documents the data persistence system of Kotatsu, explaining the database structure, repositories, and how data flows through the app. For information about higher-level business logic and use cases, see the [Domain Layer](domain-layer.md).

## Overview

The data layer in Kotatsu implements the Repository pattern, providing a clean API for accessing and manipulating manga data while abstracting away the underlying storage mechanisms. It consists of:

1. A Room database with tables for manga, tags, history, favorites, and other entities
2. Repository classes that handle specific data domains
3. Entity mapping between database models and domain models
4. Support for external data sources through content providers

`TODO diagram`

## Database Structure
### Core Database

The Kotatsu app uses Room as its database abstraction layer. The main database class `MangaDatabase` is defined with the current version 26 and contains multiple tables represented by entities.

``` mermaid
classDiagram
    class MangaDatabase
    MangaDatabase : +getHistoryDao()
    MangaDatabase : +getTagsDao()
    MangaDatabase : +getMangaDao()
    MangaDatabase : +getFavouritesDao()
    MangaDatabase : +getPreferencesDao()
    MangaDatabase : +getFavouriteCategoriesDao()
    MangaDatabase : +getTracksDao()
    MangaDatabase : +getTrackLogsDao()
    MangaDatabase : +getSuggestionDao()
    MangaDatabase : +getBookmarksDao()
    MangaDatabase : +getScrobblingDao()
    MangaDatabase : +getSourcesDao()
    MangaDatabase : +getStatsDao()
    MangaDatabase : +getLocalMangaIndexDao()
    MangaDatabase : +getChaptersDao()
```

### Database Entities

The database consists of the following main entities:

`TODO diagram`

## Repostitory Pattern Implementation

The data layer implements the repository pattern to abstract the data sources from the rest of the application. This provides a clean API for the domain layer to interact with the data layer, without needing to know the underlying storage mechanisms.

`TODO diagram`

### Key Repositiories
#### MangaSourcesRepository

The `MangaSourcesRepository` manages manga sources (parsers for different manga websites) and their configurations.

``` kotlin
@Singleton
class MangaSourcesRepository @Inject constructor(
    @LocalizedAppContext private val context: Context,
    private val db: MangaDatabase,
    private val settings: AppSettings,
) {
    // Methods for managing sources, their availability, and configurations
}
```

Key responsibilities:

* Track enabled/disabled sources
* Manage source sorting and pinning
* Observe changes to sources
* Handle external manga sources (plugins)

#### FavouritesRepository

The `FavouritesRepository` handles the user's favorite manga and their organization into categories.

``` kotlin
@Reusable
class FavouritesRepository @Inject constructor(
    private val db: MangaDatabase,
    private val localObserver: LocalFavoritesObserver,
) {
    // Methods for managing favorites and categories
}
```

Key responsibilities:

* Add/remove manga from favorites
* Create and manage favorite categories
* Observe changes to favorites

#### HistoryRepository

The `HistoryRepository` manages the user's reading history and progress.

``` kotlin
@Reusable
class HistoryRepository @Inject constructor(
    private val db: MangaDatabase,
    private val settings: AppSettings,
    private val scrobblers: Set<@JvmSuppressWildcards Scrobbler>,
    private val mangaRepository: MangaDataRepository,
    private val localObserver: HistoryLocalObserver,
    private val newChaptersUseCaseProvider: Provider<CheckNewChaptersUseCase>,
) {
    // Methods for tracking and querying reading history
}
```

Key responsibilities:

* Track reading progress
* Record viewed chapters and pages
* Query reading history
* Support filtering and sorting of history

#### MangaDataRepository

The `MangaDataRepository` is the core repository for manga data operations.

``` kotlin
@Reusable
class MangaDataRepository @Inject constructor(
    private val db: MangaDatabase,
    private val resolverProvider: Provider<MangaLinkResolver>,
    private val appShortcutManagerProvider: Provider<AppShortcutManager>,
) {
    // Methods for manga data operations
}
```

Key responsibilities:

* Store and retrieve manga details
* Manage manga reader preferences
* Handle manga chapters and pages
* Map between entity and domain models

#### ExternalMangaRepository

The `ExternalMangaRepository` handles external manga sources through content providers.

``` kotlin
class ExternalMangaRepository(
    contentResolver: ContentResolver,
    override val source: ExternalMangaSource,
    cache: MemoryContentCache,
) : CachingMangaRepository(cache) {
    // Methods for interacting with external content providers
}
```

Key responsibilities:

* Access manga from external sources
* Implement the necessary interfaces for manga repositories
* Integrate with the Android content provider system

## Data Access Objects (DAOs)

The DAOs provide methods for accessing and manipulating data in the database. Each DAO corresponds to a specific domain area and is implemented using Room's annotations.

`TODO diagram`

## Data Flow

The following diagram illustrates the data flow between different components of the data layer and how they interact with other layers.

## Entity Mapping

Entity mapping is a key component of the data layer, translating between database entities and domain models. This mapping is primarily implemented through extension functions in `EntityMapping.kt`.

`TODO diagram`

Example mapping functions:

``` kotlin
// From entity to domain model
fun TagEntity.toMangaTag() = MangaTag(
    key = this.key,
    title = this.title.toTitleCase(),
    source = MangaSource(this.source),
)

// From domain model to entity
fun MangaTag.toEntity() = TagEntity(
    title = title,
    key = key,
    source = source.name,
    id = "${key}_${source.name}".longHashCode(),
    isPinned = false, // for future use
)
```

## Backup and Serialization

The data layer includes functionality for serializing and deserializing data for backup purposes, implemented in `JsonSerializer.kt` and `JsonDeserializer.kt`.

`TODO diagram`

## Sort Orders and Filtering

The data layer supports various sorting and filtering options for manga lists, implemented through the `ListSortOrder` and `ListFilterOption` enums.

### Sort Orders

`TODO diagram`

Sort orders are used in queries to determine how manga lists are ordered:

``` kotlin
private fun getOrderBy(sortOrder: ListSortOrder) = when (sortOrder) {
    ListSortOrder.RATING -> "manga.rating DESC"
    ListSortOrder.NEWEST -> "favourites.created_at DESC"
    ListSortOrder.OLDEST -> "favourites.created_at ASC"
    ListSortOrder.ALPHABETIC -> "manga.title ASC"
    ListSortOrder.ALPHABETIC_REVERSE -> "manga.title DESC"
    ListSortOrder.NEW_CHAPTERS -> "IFNULL((SELECT chapters_new FROM tracks WHERE tracks.manga_id = manga.manga_id), 0) DESC"
    ListSortOrder.PROGRESS -> "IFNULL((SELECT percent FROM history WHERE history.manga_id = manga.manga_id), 0) DESC"
    ListSortOrder.UNREAD -> "IFNULL((SELECT percent FROM history WHERE history.manga_id = manga.manga_id), 0) ASC"
    ListSortOrder.LAST_READ -> "IFNULL((SELECT updated_at FROM history WHERE history.manga_id = manga.manga_id), 0) DESC"
    ListSortOrder.LONG_AGO_READ -> "IFNULL((SELECT updated_at FROM history WHERE history.manga_id = manga.manga_id), 0) ASC"
    ListSortOrder.UPDATED -> "IFNULL((SELECT last_chapter_date FROM tracks WHERE tracks.manga_id = manga.manga_id), 0) DESC"
    else -> throw IllegalArgumentException("Sort order $sortOrder is not supported")
}
```

## Database Migration
The database schema evolves over time, and migrations are used to update the schema without losing data. Migrations are implemented in the migrations package and are applied when the database is created.

``` kotlin
fun getDatabaseMigrations(context: Context): Array<Migration> = arrayOf(
    Migration1To2(),
    Migration2To3(),
    // ... many more migrations
    Migration25To26(),
)

fun MangaDatabase(context: Context): MangaDatabase = Room
    .databaseBuilder(context, MangaDatabase::class.java, "kotatsu-db")
    .addMigrations(*getDatabaseMigrations(context))
    .addCallback(DatabasePrePopulateCallback(context.resources))
    .build()
```

Example of a migration:

``` kotlin
class Migration25To26 : Migration(25, 26) {

    override fun migrate(db: SupportSQLiteDatabase) {
        db.execSQL("ALTER TABLE sources ADD COLUMN cf_state INTEGER NOT NULL DEFAULT 0")
        db.execSQL("ALTER TABLE preferences ADD COLUMN title_override TEXT DEFAULT NULL")
        db.execSQL("ALTER TABLE preferences ADD COLUMN cover_override TEXT DEFAULT NULL")
        db.execSQL("ALTER TABLE preferences ADD COLUMN content_rating_override TEXT DEFAULT NULL")
        db.execSQL("ALTER TABLE favourites ADD COLUMN pinned INTEGER NOT NULL DEFAULT 0")
        db.execSQL("ALTER TABLE tags ADD COLUMN pinned INTEGER NOT NULL DEFAULT 0")
    }
}
```

## Cleanup and Maintenance

The data layer includes mechanisms for cleaning up and maintaining the database, such as garbage collection for deleted entries and cleanup of local manga files.

``` kotlin
// Cleanup in MangaDao
@Query(
    """
    DELETE FROM manga WHERE NOT EXISTS(SELECT * FROM history WHERE history.manga_id == manga.manga_id) 
        AND NOT EXISTS(SELECT * FROM favourites WHERE favourites.manga_id == manga.manga_id)
        AND NOT EXISTS(SELECT * FROM bookmarks WHERE bookmarks.manga_id == manga.manga_id)
        AND NOT EXISTS(SELECT * FROM suggestions WHERE suggestions.manga_id == manga.manga_id)
        AND NOT EXISTS(SELECT * FROM scrobblings WHERE scrobblings.manga_id == manga.manga_id)
        AND NOT EXISTS(SELECT * FROM local_index WHERE local_index.manga_id == manga.manga_id)
        AND manga.manga_id NOT IN (:idsToKeep)
    """,
)
abstract suspend fun cleanup(idsToKeep: Set<Long>)

// LocalStorageCleanupWorker
@HiltWorker
class LocalStorageCleanupWorker @AssistedInject constructor(
    @Assisted appContext: Context,
    @Assisted params: WorkerParameters,
    private val settings: AppSettings,
    private val localMangaRepository: LocalMangaRepository,
    private val dataRepository: MangaDataRepository,
    private val deleteReadChaptersUseCase: DeleteReadChaptersUseCase,
) : CoroutineWorker(appContext, params) {

    override suspend fun doWork(): Result {
        if (settings.isAutoLocalChaptersCleanupEnabled) {
            deleteReadChaptersUseCase.invoke()
        }
        return if (localMangaRepository.cleanup()) {
            dataRepository.cleanupLocalManga()
            Result.success()
        } else {
            Result.retry()
        }
    }
}
```