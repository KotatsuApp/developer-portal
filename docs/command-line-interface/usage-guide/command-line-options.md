# Command Line Options

This document provides a comprehensive reference of all command line options available in kotatsu-dl, a cross-platform manga downloader. It explains the purpose, syntax, and behavior of each option. For information about supported manga sources, see [Supported Sources](supported-sources.md). For details about output formats, see [Output Formats](output-formats.md).

## Basic Usage

The basic syntax for using kotatsu-dl is:

```
kotatsu-dl [<options>] <link>
```

Where `<link>` is a required argument representing the URL to the manga you want to download, and `[<options>]` are optional parameters that modify the behavior of the program.

## Command Line Arguments
### Link (Required)

```
kotatsu-dl <link>
```

The `<link>` argument is required and must be a direct URL to the manga copied from a browser. This link is used to identify the manga source and locate the manga to download.

## Command Line Options
### Destination

```
--dest <path>, --destination <path>
```

Specifies the output file or directory path where the downloaded manga will be saved. If not provided, the current directory is used.

The path supports using `~` to reference the user's home directory, which will be automatically expanded.

### Format

```
--format <format>
```

Specifies the output format for the downloaded manga. Valid values are:

* `cbz` - Comic Book ZIP format
* `zip` - Standard ZIP archive
* `dir` - Regular directory structure

If not specified, the format is determined automatically:

1. Based on the destination file extension if provided
2. For manga with 5 or fewer chapters, CBZ format is used by default
3. For manga with more than 5 chapters, DIR format is used by default

### Parallelism

```
-j <number>, --jobs <number>
```

Sets the number of parallel download jobs. This controls how many pages or chapters are downloaded simultaneously.

* Default: `4`
* Allowed range: `1` to `10`

Setting a higher number may increase download speed but also increases the risk of being rate-limited by the manga source server.

### Throttle

```
--throttle
```

Enables throttling to slow down the download process. This is useful to avoid being detected and blocked by manga source servers that have anti-scraping measures.

* Default: `false` (throttling is disabled)

When enabled, the downloader will introduce delays between requests to appear more like a normal user browsing the site.

### Chapters Range

```
--chapters <range>
```

Specifies which chapters to download. The range can be:

* Single numbers: `1,3,5`
* Ranges: `1-5`
* Combination of both: `1-5,7,9-11`
* `all`: Download all available chapters (default behavior)

If this option is not provided and there are multiple chapters, the program will prompt the user to enter a range interactively.

### Verbose

```
-v, --verbose
```

Enables verbose output, showing more detailed information during the download process. This is useful for troubleshooting or monitoring the download progress in detail.

* Default: `false` (standard output)

### Sources List

```
--sources
```

Displays a list of all supported manga sources and exits. This is an "eager" option, meaning it will be processed immediately when encountered and will prevent further command processing.

The list is grouped by language and includes information about whether a source contains adult content or is currently broken.

### Help

```
-h, --help
```

Displays help information about all available commands and options, then exits.

## Examples
Here are some examples of how to use kotatsu-dl with various command line options:
### Basic Download (All Chapters)

```
kotatsu-dl https://example-manga-site.com/manga/title
```

### Download to Specific Location

```
kotatsu-dl --dest ~/Downloads/my-manga https://example-manga-site.com/manga/title
```

### Download Specific Chapters in CBZ Format

```
kotatsu-dl --format cbz --chapters 1-5,7,10 https://example-manga-site.com/manga/title
```

### Throttled Download with Increased Verbosity

```
kotatsu-dl --throttle -v https://example-manga-site.com/manga/title
```

### Display All Supported Sources

```
kotatsu-dl --sources
```

## Common Option Combinations
The following table summarizes common combinations of options for specific use cases:

| Use Case                               | Command Line Options           |
| -------------------------------------- | ------------------------------ |
| Fast download (low risk of blocking  ) | `-j 8`                         |
| Safer download (high risk of blocking) | `--throttle -j 2`              |
| Archive for e-readers                  | `--format cbz`                 |
| Reading on computer                    | `--format dir`                 |
| Debugging connection issues            | `-v --throttle`                |
| Selected chapters to archive           | `--format cbz --chapters 1-10` |

## Interactive Features
If the `--chapters` option is not provided and the manga has multiple chapters, the program will prompt the user to enter a range interactively. This is done through the command line interface.

Similarly, if a manga has multiple branches (e.g., different translations or editions), the user will be prompted to select which branch to download.