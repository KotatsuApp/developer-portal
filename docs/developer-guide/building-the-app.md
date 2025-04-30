# Building the App

This document provides step-by-step instructions for setting up the development environment and building the Kotatsu manga reader app. It covers the necessary prerequisites, project setup, build configuration, and various build variants.

For information about adding translations, see [Contributing Translations](contributing-translations.md). For instructions on adding new manga sources, see [Adding New Manga Sources](adding-new-manga-sources.md).

## Prerequisites
Before building Kotatsu, ensure you have the following tools installed:

| Tool           | Required Version   | Notes                                             |
| -------------- | ------------------ | ------------------------------------------------- |
| JDK            | 11                 | The app uses Java 11 language features            |
| Android Studio | Latest recommended | Or IntelliJ IDEA with Android plugins             |
| Git            | Any recent version | For cloning the repository                        |
| Gradle         | 8.12+              | Automatically provided by the Gradle wrapper      |

## Project Structure
### Build System Overview

The Kotatsu app uses Gradle as its build system with Kotlin DSL build scripts. The project follows a standard Android application structure with several key build files.

``` mermaid
flowchart TB
    subgraph ide1 [Build System Files]
    settings.gradle-->build.gradle
    gradle/libs.versions.toml-->build.gradle
    gradle-wrapper.properties-->build.gradle
    gradle/libs.versions.toml-->app/build.gradle
    build.gradle-->app/build.gradle
    gradle.propeties-->app/build.gradle
    end
```

### Project Configuration Files

``` mermaid
flowchart TB
    subgraph ide1 [Repositories]
    settings.gradle-- Repository config -->id1[Remote repositories]
    end
    subgraph ide2 [Dependency Management]
    libs.versions.toml-- Versions & dependencies -->app/build.gradle
    build.gradle-- Plugin application -->app/build.gradle
    id1-- Dependncy resolution -->app/build.gradle
    end
    subgraph ide3 [Build Properties]
    gradle.properties-- Global settings -->app/build.gradle
    app/build.gradle-- Defines -->id2[Build types]
    app/build.gradle-- Configures -->id3[Build features]
    end
```

## Getting Started
### Cloning the Repository
To get started, clone the Kotatsu repository:

```
git clone https://github.com/KotatsuApp/Kotatsu.git
cd Kotatsu
```

### Opening the Project
Open the project in Android Studio:

1. Launch Android Studio
2. Select "Open an existing project"
3. Navigate to the cloned Kotatsu directory and click "Open"

Android Studio will sync the project with Gradle files automatically.

## Build Configuration
### Build Variants
Kotatsu has three build variants, each serving a different purpose:

| Build Type | Purpose                 | Configuration                                                      |
| ---------- | ----------------------- | ------------------------------------------------------------------ |
| debug      | Development and testing | Adds `.debug` suffix to package name                               |
| release    | Production releases     | Enables minification and shrinks resources                         |
| nightly    | Automated daily builds  | Based on release but with `.nightly` suffix and dynamic versioning |

### Key Build Features
The app uses several Gradle features and plugins:

| Feature/Plugin | Purpose                                       |
| -------------- | --------------------------------------------- |
| viewBinding    | Enables type-safe view binding                |
| buildConfig    | Generates BuildConfig class                   |
| parcelize      | Implements Parcelable interface automatically |
| hilt           | Dependency injection                          |
| ksp            | Kotlin Symbol Processing for code generation  |
| room           | Database ORM                                  |

## Dependencies