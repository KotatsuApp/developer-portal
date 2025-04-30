# Installation & Setup

This document covers the installation and configuration process for kotatsu-dl, a cross-platform manga downloader. For information about using the tool after installation, see Usage Guide.

## Prerequisites
Before installing kotatsu-dl, ensure your system meets the following requirements:

* **Java Runtime Environment (JRE)**: Version 17 or later is required
* **Internet connection**: Required for downloading manga from supported sources
* **Storage space**: Sufficient space to store downloaded manga

## Installation Methods
kotatsu-dl can be installed through different methods depending on your operating system.

### Pre-built Binary (All Platforms)

1. Download the latest release JAR file from the 
[GitHub releases page](https://github.com/KotatsuApp/kotatsu-dl/releases/latest)
2. No additional installation is required - the JAR file contains all necessary dependencies
3. Run the application using the Java command:
```
java -jar kotatsu-dl.jar
```

### ArchLinux Package
For ArchLinux users, kotatsu-dl is available in the AUR (Arch User Repository):

```
yay -S kotatsu-dl-git
```

Once installed, the `kotatsu-dl` command will be available system-wide.

### Building from Source
For developers or users who want the latest features, building from source is an option:

1. Clone the repository:
```
git clone https://github.com/KotatsuApp/kotatsu-dl.git
```
2. Navigate to the project directory:
```
cd kotatsu-dl
```
3. Build using Gradle:
```
./gradlew shadowJar
```
4. The compiled JAR will be available in the `build/libs` directory

## Verification
After installation, verify kotatsu-dl is working correctly by running a simple command:

``` bash
# If using the JAR file directly
java -jar kotatsu-dl.jar --help

# If installed via AUR
kotatsu-dl --help
```

You should see the help output displaying available options.

## Configuration
kotatsu-dl is primarily configured through command-line arguments. There are no separate configuration files to set up.

## Troubleshooting
### Common Installation Issues
| Issue                 | Possible Cause                       | Solution                                              |
| --------------------- | ------------------------------------ | ----------------------------------------------------- |
| "Java not found"      | Java is not installed or not in PATH | Install Java 17+ and ensure it's in your system PATH  |
| Outdated Java version | Using Java version lower than 17     | Update to Java 17 or later                            |
| Permission denied     | JAR file not executable              | Add execute permission: `chmod +x kotatsu-dl.jar`     |
| AUR package error     | Build dependencies missing           | Install required build dependencies before installing |

### Checking Java Version
To verify your Java version, run:
```
java -version
```
Ensure the output shows version 17 or higher.

## Next Steps
After successful installation, refer to the Usage Guide for detailed information on how to use kotatsu-dl to download manga from various sources.