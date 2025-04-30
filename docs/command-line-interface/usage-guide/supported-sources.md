# Supported Sources

## Overview
This page documents the manga sources supported by kotatsu-dl. The application integrates with a wide variety of manga sources across multiple languages through the `kotatsu-parsers` library. For information about output formats, see [Output Formats](output-formats.md).

## Source Organization
kotatsu-dl organizes its supported manga sources by language (locale). Each source provides access to a specific manga website and is associated with a content type. The application uses the `MangaParserSource` enum defined in the `kotatsu-parsers` library to represent these sources.

## Viewing Available Sources
To view a list of all supported manga sources, you can use the `--sources` command line option:

``` bash
java -jar kotatsu-dl.jar --sources
```

This command displays a list of all available sources, organized by language, along with their domains and any special flags (such as content type or broken status).

## Source Output Format
When displayed, each source shows the following information:

* Source name (title)
* Website domain
* Content type label (if not standard manga)
* Broken status (if applicable)

Example output format:

```
List of supported sources:
 - English
   MangaDex https://mangadex.org
   ComicExtra https://comicextra.com [Comics]
   MangaOwl https://mangaowl.net
   ...
 - Japanese
   RawKuma https://rawkuma.com
   ...
 - Russian
   Desu https://desu.me
   ...
```

## Source Types
Sources in kotatsu-dl are categorized by their content type:

| Content Type | Description              | Indicator |
| ------------ | ------------------------ | --------- |
| MANGA        | Standard manga (default) | None      |
| COMICS       | Western comics           | [Comics]  |
| HENTAI       | Adult manga content      | [Hentai]  |

Additionally, some sources may be marked as broken, indicating that they may not function correctly due to website changes or other issues.

## Technical Details
### Source Validation
When processing a manga URL, kotatsu-dl validates that the source is supported:

``` kotlin
val source = linkResolver.getSource()
if (source == null || source == MangaParserSource.DUMMY) {
    error("Unsupported manga source")
}
```

This check ensures that only supported sources are processed, avoiding potential errors with unsupported websites.

### Source Instances
Each source requires a parser instance to be created. During the source listing process, sources that fail to instantiate their parsers are skipped:

``` kotlin
val parser = try {
    newParserInstance(source)
} catch (_: Throwable) {
    continue
}
```

This ensures that only functional sources are displayed to the user.

## Adding New Sources
Adding new sources to kotatsu-dl requires modifying the `kotatsu-parsers` library, not kotatsu-dl itself. The application automatically detects and utilizes all sources provided by the parser library.