# Usage Guide

This guide provides comprehensive instructions on how to use kotatsu-dl, a command-line tool for downloading manga from various online sources. It covers basic usage, command-line options, and workflow examples. For installation instructions, please refer to [Installation & Setup](../installation-and-setup.md).

## Basic Usage
kotatsu-dl is designed to download manga using a simple command structure:

``` bash
java -jar kotatsu-dl.jar [options] <manga-url>
```

Where `<manga-url>` is a direct link to the manga you want to download, copied from your browser.

Basic example:

``` bash
java -jar kotatsu-dl.jar https://manga-site.com/manga/title
```

The tool will automatically:

1. Identify the manga source
2. Retrieve manga information
3. Present a chapter selection interface
4. Download selected chapters
5. Save them in the appropriate format

## Command Line Options
kotatsu-dl provides several options to customize the download process:

| Option                    | Description                                                 | Default               |
| ------------------------- | ----------------------------------------------------------- | --------------------- |
| `--dest`, `--destination` | Output file or directory path                               | Current directory     |
| `--format`                | Output format (cbz, zip, dir)                               | Auto-detected         |
| `-j`, `--jobs`            | Number of parallel download jobs                            | 4                     |
| `--throttle`              | Slow down downloads to avoid IP blocks                      | Off                   |
| `--chapters`              | Numbers of chapters to download (e.g., "1-4,8,11" or "all") | Interactive selection |
| `-v`, `--verbose`         | Show more information                                       | Off                   |
| `--dest`, `--destination` | Output file or directory path                               | -                     |
| `-h`, `--help`            | Show help message and exit                                  | -                     |

For a more detailed explanation of each option, see [Command Line Options](command-line-options.md).

## Output Formats
kotatsu-dl supports multiple output formats:

1. CBZ - Comic book archive format (recommended for smaller manga)
2. ZIP - Standard ZIP archive
3. DIR - Plain directory structure (recommended for larger manga)

If no format is specified, kotatsu-dl will:

* Use the extension from the destination filename if provided
* Choose CBZ for manga with 5 or fewer chapters
* Choose DIR for manga with more than 5 chapters

For more details on output formats, see [Output Formats](output-formats.md).

## Using Chapter Selection
kotatsu-dl provides flexible chapter selection options:

### Interactive Selection
By default, if no `--chapters` option is provided, kotatsu-dl will prompt you to input which chapters you want to download.

Example interactive prompt:

```
==> Chapters to download (e.g. "1-4,8,11" or empty for all):
==> 
```

Pressing ++enter++ without input will download all chapters.

### Command-Line Selection
You can specify chapters using the `--chapters` option:

``` bash
java -jar kotatsu-dl.jar --chapters "1-4,8,11" https://manga-site.com/manga/title
```

This will download chapters 1, 2, 3, 4, 8, and 11.

Chapter selection syntax:

* Single numbers: `1,3,5`
* Ranges: `1-5`
* Combinations: `1-3,5,7-9`
* All chapters: `all`

## Performance Optimization
kotatsu-dl provides options to optimize download performance:

### Parallel Downloads
Use the `-j` or `--jobs` option to control how many parallel downloads are performed:

``` bash
java -jar kotatsu-dl.jar -j 8 https://manga-site.com/manga/title
```

Higher values may improve speed but could also increase the risk of being rate-limited by the source website.

### Throttling
If you're experiencing issues with websites blocking your IP address, use the `--throttle` option:

``` bash
java -jar kotatsu-dl.jar --throttle https://manga-site.com/manga/title
```

This will slow down the download process to avoid triggering anti-scraping protections.

## Viewing Supported Sources
To see which manga sources are supported by kotatsu-dl, use the `--sources` option:

``` bash
java -jar kotatsu-dl.jar --sources
```

This will display a list of all supported sources grouped by language, including domain information and any special content types or status indicators.

## Example Commands
Here are some example commands for common use cases:

### Download a manga to a specific location:

``` bash
java -jar kotatsu-dl.jar --dest "/home/user/manga" https://manga-site.com/manga/title
```

### Download specific chapters in CBZ format:

``` bash
java -jar kotatsu-dl.jar --format cbz --chapters "1-5,10,15-20" https://manga-site.com/manga/title
```

### Throttled download with verbose output:

``` bash
java -jar kotatsu-dl.jar --throttle -v https://manga-site.com/manga/title
```

### Maximum parallel downloads:

``` bash
java -jar kotatsu-dl.jar -j 10 https://manga-site.com/manga/title
```

## Troubleshooting
### Common Issues

1. Unsupported manga source: Make sure the website is in the supported sources list. Check with `--sources`.
2. No chapters found: Verify the URL is correct and the manga actually has chapters available.
3. Download failures: Some sources may have anti-scraping measures. Try using the `--throttle` option.
4. High memory usage: For very large manga, consider using the dir output format instead of `cbz` or `zip`.
