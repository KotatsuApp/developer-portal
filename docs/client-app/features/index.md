---
title: Features
description: This page provides an overview of the main features available in the Kotatsu manga reader application.
---

# Features

This page provides an overview of the main features available in the Kotatsu manga reader application. It highlights the key functionality areas that users can access and how they interconnect, giving a broad understanding of what Kotatsu offers. For more detailed information about specific features, please refer to their dedicated sections: Manga Reader, Manga Details, Library Management, Tracking and Updates, and Manga Sources.

## Core Features Overview
Kotatsu is a comprehensive manga reader application that offers a rich set of features for discovering, reading, and managing manga content:

| Feature Category     | Description                                                                                                                            |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Reading Experience   | Multiple reading modes (Standard, Right-to-Left, Vertical, Webtoon), page navigation controls, zoom options, and visual customizations |
| Content Sources      | Support for numerous online manga sources, local storage, and file imports                                                             |
| Library Management   | Favorites with categories, reading history, bookmarks, and downloads                                                                   |
| Tracking and Updates | New chapter notifications, reading progress tracking, and content synchronization                                                      |
| Discovery            | Search functionality, suggestions system, and browsing by source                                                                       |
| Customization        | Extensive settings for appearance, behavior, and performance optimization                                                              |
| Accessibility        | Multiple languages support and various content display options                                                                         |

## Feature Architecture Overview

`TODO diagram`

## Reading Experience

The reading experience is a central feature of Kotatsu, offering multiple customizable reading modes to suit various manga styles and user preferences.

### Reader Modes
Kotatsu supports four distinct reading modes:

1. **Standard** - Traditional left-to-right page reading
2. **Right-to-Left** - For manga with traditional Japanese reading direction
3. **Vertical** - Vertical scrolling through individual pages
4. **Webtoon** - Continuous vertical scrolling, optimized for webtoons

The app can automatically detect the appropriate mode based on content type, or users can manually select their preferred mode.

### Reader Customization

The reader offers extensive customization options for an optimal reading experience:

* **Zoom controls**: Different scaling modes (Fit Center, Fit Height, Fit Width, Keep at Start)
* **Navigation**: Page-turning through taps, volume buttons, or on-screen controls
* **Visual settings**: Background color, brightness, contrast, color inversion, grayscale
* **Layout options**: Information bar, page numbers, and reader orientation
* **Performance settings**: Memory optimization and page preloading

Users can customize the tap zones on the screen to perform different actions (switch pages, open menus, etc.) through a configurable touch grid system.

## Content Sources System

Kotatsu provides access to manga content from multiple sources, allowing users to discover and read a wide variety of manga.

### Online Manga Sources

The app includes support for numerous online manga sources, which can be enabled or disabled according to user preference. Each source is represented by the MangaSource interface and can provide metadata and content.

`TODO diagram`

### Local Storage

In addition to online sources, Kotatsu supports reading manga from local storage:

* **Local files**: Import manga from ZIP/CBZ archives or folders of images
* **Downloaded content**: Save manga for offline reading
* **Cross-reference**: Local content can be linked with online sources for metadata

## Library Management

Kotatsu provides several ways to organize and manage manga collections for easy access and continuation of reading.

### Favorites System
Users can add manga to favorites and organize them into custom categories:

* Create, edit, and delete favorite categories
* Add manga to multiple categories
* Sort and filter within favorites
* Pin favorites to the app shelf for quick access

### History Tracking
The app automatically tracks reading progress and history:

* Continue reading from where you left off
* View recently read manga
* Track completion percentage for each manga
* Optional incognito mode to prevent history recording

### Bookmarks
Bookmarks allow users to mark specific pages within chapters:

* Add bookmarks to important or favorite scenes
* Quickly navigate to bookmarked pages
* Manage and delete bookmarks

## Tracking and Updates
Kotatsu provides features to help users stay updated with new content and track their reading progress.

### New Chapter Notifications
The app can automatically check for updates to manga in a user's favorites:

* Background checking for new chapters
* Customizable notification settings
* Feed of recently updated manga

### Reading Progress Tracking
The app tracks reading progress in several ways:

* Percentage read of each manga
* Chapter-level tracking
* Optional synchronization across devices

## Content Discovery
Kotatsu offers multiple ways to discover new manga content.

### Search System
The app provides powerful search capabilities:

* Search across all enabled sources
* Search by title, author, or tags
* Search history for quick access to previous queries
* Source-specific advanced filtering

### Suggestions System
Kotatsu includes a local suggestion system that recommends manga based on reading history and preferences:

* Personalized recommendations based on reading patterns
* Ability to exclude specific genres or NSFW content
* All processing happens locally on the device for privacy

## Customization and Settings
Kotatsu offers extensive customization options to tailor the app to individual preferences:

### Application Settings

`TODO diagram`

### User Interface Customization
The app offers several UI customization options:

* Multiple themes (Light, Dark, AMOLED black)
* Various color schemes
* Customizable main screen sections
* Grid size adjustments for manga lists
* Optional navigation bar labels

### Reader Customization
In addition to reading modes, users can customize the reader experience:

* Page animations
* Background colors
* Screen orientation
* Tap actions
* Information display options

## Accessibility and Internationalization
Kotatsu is designed to be accessible to users around the world:

### Multi-language Support
The app includes translations for numerous languages:

* Interface language can be set independently from system language
* Support for various reading directions (left-to-right, right-to-left)
* Manga sources in multiple languages

### Visual Accessibility
Various features enhance visual accessibility:

* Adjustable text and UI sizing
* High contrast options
* Color inversion and grayscale modes for reading

## Data Management and Backup
The app includes features for managing data and ensuring continuity:

* Create and restore backups of favorites and history
* Periodic automated backups
* Data synchronization across devices
* Storage management tools

