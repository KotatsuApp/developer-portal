---
title: Build System
description: This document provides an overview of the Gradle-based build system used in the Kotatsu.
---

# Build System

This document provides an overview of the Gradle-based build system used in the Kotatsu Manga Reader application. It covers the configuration files, build types, dependency management, and core plugins that define how the application is built, packaged, and deployed.

For information about setting up the development environment and building the app, see Building the App.

## 1. Overview of the Build System

The Kotatsu app uses Gradle as its build automation tool with a modern Kotlin DSL configuration approach. The build system is responsible for compiling the application, managing dependencies, configuring build variants, and handling the build process for different environments.

`TODO diagram`

## 2. Core Gradle Configuration Files

The build system consists of several configuration files that work together to define the build process:

| File                                       | Purpose                                                                                                      |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| `app/build.gradle`                         | Main application module configuration, including Android-specific settings, dependencies, and build variants |
| `build.gradle`                             | Root project build file that applies plugins to the project                                                  |
| `gradle/libs.versions.toml`                | Version catalog that centralizes dependency management                                                       |
| `settings.gradle`                          | Project settings including repository configurations and module inclusion                                    |
| `gradle.properties`                        | Global Gradle and Android configuration properties                                                           |
| `gradle/wrapper/gradle-wrapper.properties` | Gradle version configuration                                                                                 |

## 3. Build Types and Variants

Kotatsu defines three main build types, each serving a different purpose in the development lifecycle:

`TODO diagram`

### 3.1. Debug Build

The debug build is designed for development and testing, with:

* Application ID suffix .debug to allow installation alongside production versions
* Debug symbols and logging enabled
* No code shrinking or obfuscation

### 3.2. Release Build

The release build is optimized for production:

* Code minification enabled
* Resource shrinking enabled
* ProGuard rules applied for optimization
* No application ID suffix (uses the base application ID)

### 3.3. Nightly Build

The nightly build is a special variant based on the release build:

* Inherits release build configuration
* Uses application ID suffix `.nightly`
* Dynamic version code and name based on build date
* Automatic versioning using `LocalDateTime.now()`

## 4. Dependency Management
### 4.1. Version Catalog System

Kotatsu uses Gradle's Version Catalog (`libs.versions.toml`) to centralize dependency management:

`TODO diagram`

The version catalog contains:

* Version declarations for all dependencies
* Library declarations with references to version entries
* Plugin declarations for use across modules

### 4.2. Key Dependencies

The application relies on several key dependency groups:

| Category      | Dependencies                                                        |
| ------------- | ------------------------------------------------------------------- |
| Android Core  | AndroidX Core, AppCompat, Material Components                       |
| UI Components | RecyclerView, ViewPager2, ConstraintLayout, SwipeRefreshLayout      |
| Architecture  | Lifecycle, Room, Hilt (Dependency Injection)                        |
| Networking    | OkHttp, Okio                                                        |
| Image Loading | Coil, SSIV (SubsamplingScaleImageView), AVIF support                |
| Coroutines    | Kotlin Coroutines                                                   |
| Others        | ACRA (crash reporting), Markwon (Markdown), LeakCanary (debug only) |

### 4.3. External Parser Dependency

A unique aspect of the dependency management is the handling of the Kotatsu Parsers library:

``` groovy
def parsersVersion = libs.versions.parsers.get()
if (System.properties.containsKey('parsersVersionOverride')) {
    // usage:
    // -DparsersVersionOverride=$(curl -s https://api.github.com/repos/kotatsuapp/kotatsu-parsers/commits/master -H "Accept: application/vnd.github.sha" | cut -c -10)
    parsersVersion = System.getProperty('parsersVersionOverride')
}
implementation("com.github.KotatsuApp:kotatsu-parsers:$parsersVersion") {
    exclude group: 'org.json', module: 'json'
}
```

This allows overriding the parsers library version from command line parameters, facilitating development with the latest parser changes.

## 5. Plugins and Build Features
### 5.1.

`TODO diagram`

### 5.2. Build Features

Several build features are enabled to support modern Android development:

``` groovy
buildFeatures {
    viewBinding true
    buildConfig true
}
```

* **viewBinding**: Enables type-safe binding to XML layout files
* **buildConfig**: Generates the BuildConfig class with build-specific constants

### 5.3. Java and Kotlin Configuration
The application uses Java 11 compatibility with specific Kotlin compiler options:

``` groovy
compileOptions {
    coreLibraryDesugaringEnabled true
    sourceCompatibility JavaVersion.VERSION_11
    targetCompatibility JavaVersion.VERSION_11
}
kotlinOptions {
    jvmTarget = JavaVersion.VERSION_11.toString()
    freeCompilerArgs += [
        // Kotlin experimental features and opt-ins
        // ...
    ]
}
```

The `coreLibraryDesugaringEnabled` option allows using Java 8+ APIs on older Android devices through desugaring.

## 6. Testing Configuration
The build system includes comprehensive testing configurations:

``` groovy
testOptions {
    unitTests.includeAndroidResources true
    unitTests.returnDefaultValues false
    kotlinOptions {
        freeCompilerArgs += ['-opt-in=org.kotatsu.kotatsu.parsers.InternalParsersApi']
    }
}
```

Special configurations are applied to unit tests, including access to Android resources and special Kotlin compiler options for test code.

## 7. Repository Configuration

The project uses multiple repositories for dependency resolution:

`TODO diagram`

JitPack is particularly important as it provides access to GitHub-hosted libraries like the Kotatsu Parsers.

## 8. Gradle Properties Configuration
The `gradle.properties` file contains global settings that affect the build process:

| Property                      | Value                                               | Purpose                                           |
| ----------------------------- | --------------------------------------------------- | ------------------------------------------------- |
| `android.enableJetifier`      | `false`                                             | Legacy support library compatibility (disabled)   |
| `android.useAndroidX`         | `true`                                              | Use AndroidX libraries instead of Support Library |
| `android.nonTransitiveRClass` | `true`                                              | R class optimization                              |
| `android.enableR8.fullMode`   | `true`                                              | Enable advanced R8 optimizations                  |
| `kotlin.code.style`           | `official`                                          | Use the official Kotlin code style                |
| `org.gradle.jvmargs`          | `-Xmx1536M -Dkotlin.daemon.jvm.options="-Xmx1536M"` | JVM memory configuration                          |

## 9. Application Metadata
The basic application configuration is defined in the Android block of the app build.gradle:

``` groovy
android {
    compileSdk = 35
    buildToolsVersion = '35.0.0'
    namespace = 'org.koitharu.kotatsu'

    defaultConfig {
        applicationId 'org.koitharu.kotatsu'
        minSdk = 21
        targetSdk = 35
        versionCode = 1009
        versionName = '8.1.3'
        // ...
    }
    // ...
}
```

This defines critical application properties such as package name, SDK versions, and app versioning.