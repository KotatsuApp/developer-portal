# Output System

The Output System in kotatsu-dl is responsible for saving downloaded manga in various formats to the local filesystem. This component handles the creation of output files or directories, organization of manga content, and management of metadata. For information about the overall download process, see [Download Process](download-process.md).

## Overview
The Output System provides a flexible framework for storing downloaded manga content in different formats. It abstracts the output mechanism through a base class that allows for various implementations while maintaining a consistent interface for the download process.

## Output Formats
The system supports three output formats:

| Format | Description | Best Used For |
| ---------- | ---------- | ---------- |
| CBZ | Comic Book ZIP archive (renamed ZIP file) | Small manga collections, ease of reading in comic readers |
| ZIP | Standard ZIP archive | Compatibility with standard archive software |
| DIR | Directory structure with individual chapter files | Large manga collections, ability to read while downloading |

## LocalMangaOutput Base Class
`LocalMangaOutput` is the abstract base class that defines the interface for all output implementations. It implements `Closeable` to ensure proper resource handling.

### Key Methods

| Method | Purpose |
| -------- | ----- |
| `mergeWithExisting()` | Combines newly downloaded content with existing output |
| `addCover(file, ext)` | Adds the manga cover image to the output |
| `addPage(chapter, file, pageNumber, ext)` | Adds a manga page to the output |
| `flushChapter(chapter)` | Finalizes a chapter after all its pages are added |
| `finish()` | Completes the output process |
| `cleanup()` | Removes temporary files in case of errors |

### Factory Method
The static `create()` method serves as a factory that instantiates the appropriate output implementation based on:

* The destination file or directory
* The manga metadata
* The preferred format (if specified)
This method handles various scenarios:

1. Writing to an existing file/directory
2. Creating a new file/directory in an existing parent directory
3. Creating a new file/directory with its parent directories

## Directory Output Implementation
The `LocalMangaDirOutput` class implements the directory-based output format. It organizes manga content as:

```
manga_folder/
  ├── cover.jpg
  ├── index.json
  ├── chapter1.cbz
  ├── chapter2.cbz
  └── ...
```

Key features:

* Each chapter is stored as an individual CBZ file
* An index.json file contains metadata about the manga and its chapters
* Mutex synchronization ensures thread safety during concurrent operations
* Temporary files are used during writing and renamed upon successful completion

## Manga Index
The `MangaIndex` class manages metadata for directory-based manga output. It stores:

* Basic manga information (title, author, etc.)
* Cover file reference
* Chapter information and file mappings

The index is saved as a JSON file named `index.json` in the root of the manga directory. This enables applications to quickly load manga information without parsing all chapter files.

## Error Handling and Resource Management
The Output System implements robust error handling and resource management:

1. All implementations extend `Closeable` to ensure proper resource cleanup
2. Temporary files are used during writing to prevent corruption
3. Mutex locks prevent concurrent modification in multi-threaded contexts
4. `cleanup()` method handles removal of temporary files during errors
5. Files are only renamed from temporary to final after successful completion

This approach ensures that failed downloads don't leave the filesystem in an inconsistent state.