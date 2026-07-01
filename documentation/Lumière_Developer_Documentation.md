# Lumière - Complete Developer Documentation

Welcome to the comprehensive Lumière documentation. This guide is designed to give developers of all levels the knowledge they need to understand, extend, and contribute to the Lumière ecosystem. Whether you're fixing a bug, adding a new feature, or building a community plugin, this document provides an expert-level overview of the entire codebase.

**[Jump to Section](#table-of-contents)**

## Table of Contents

- [Project Overview](#project-overview)
- [Technology Stack](#technology-stack)
- [Project Architecture](#project-architecture)
- [Frontend Engine (Browser)](#frontend-engine-browser)
- [Backend Engine (Electron Main Process)](#backend-engine-electron-main-process)
- [Plugin Development](#plugin-development)
  - [Plugin Lifecycle](#plugin-lifecycle)
  - [Plugin API Reference](#plugin-api-reference)
  - [Building a Basic Plugin](#building-a-basic-plugin)
- [Profiles Engine Documentation](#profiles-engine-documentation)
- [Achievements Engine Documentation](#achievements-engine-documentation)
- [Complete Function Reference Table](#complete-function-reference-table)
- [Contribution Guide](#contribution-guide)

## Project Overview

Lumière is a premium, open-source media player built with Electron. It provides a cinematic interface for organizing, streaming, and watching local media files and online content. The application is designed with a decoupled architecture, allowing for extensive customization through a community-driven plugin system.

### Core Principles
- **User-Centric:** Simple, intuitive interface with focus on media discovery.
- **Extensible:** Full plugin API for adding new content sources and features.
- **Performant:** Hardware-accelerated video playback with automatic fallback.
- **Secure:** Sandboxed plugins and isolated session contexts for safety.

## Technology Stack

| Layer | Technology |
| :--- | :--- |
| **UI Framework** | Vanilla JavaScript, Tailwind CSS, HTML5 |
| **Desktop Engine** | Electron v42.5.2 |
| **Media Streaming** | Express.js, FFmpeg (Hardware Accelerated) |
| **Data Storage** | IndexedDB (Client), JSON files (Server) |
| **Background Processing** | Node.js `vm` module (Plugin Sandbox) |
| **External APIs** | TMDB, YTS, Custom Provider Plugins |

## Project Architecture

The Lumière ecosystem is divided into three main layers:

1.  **Frontend (Browser):** The user interface and client-side logic. Handles rendering, user input, navigation, and state management.
2.  **Preload (Context Bridge):** Securely exposes Node.js and Electron APIs to the frontend.
3.  **Backend (Main Process):** The Node.js environment managing windows, file I/O, hardware acceleration, plugin sandboxing, and HTTP streaming.

### Directory Structure

| File | Description |
|------|-------------|
| `index.html` | Main UI entry point |
| `main_process.js` | Electron main process |
| `preload.js` | Context bridge |
| `package.json` | Project dependencies |
| `constants.js` | Static data (GENRES, MENU, etc.) |
| `state.js` | Global application state |
| `db.js` | IndexedDB wrapper for profiles |
| `ui.js` | UI rendering logic |
| `navigation.js` | Mode switching, searching, pagination |
| `player.js` | Video playback controls |
| `main.js` | App initialization & global listeners |
| `library.js` | Local media scanning & import |
| `audio-engine.js` | Audio processing (Night Mode) |
| `achievements.js` | Gamification engine |
| `profile.js` | Profile management & parental controls |
| `package-lock.json` | Lockfile for dependencies |

## Frontend Engine (Browser)

The frontend is a single-page application (SPA) that manages all user interactions. The core logic is split into several key modules.

### Global State (`state.js`)

The `State` object is the single source of truth for the application's condition. It holds everything from the active profile and current viewing mode to the list of installed plugins and playback status.

| Property | Type | Description |
|----------|------|-------------|
| `activeProfile` | `object` | The currently selected user profile. |
| `mode` | `string` | The current browsing mode (`'tv'`, `'movie'`, `'anime'`). |
| `selectedItem` | `object` | The media item currently open in the modal. |
| `isPlaying` | `boolean` | Indicates if a video is currently playing. |
| `plugins` | `array` | Array of installed plugin metadata. |
| `volume` | `number` | Current audio volume level (0.0 to 1.0). |

---

### UI Rendering (`ui.js`)

Responsible for generating all HTML elements. It uses a component-based approach to render navigation, grid views, content rows, and hero sections.

| Function | Description |
|----------|-------------|
| `renderNav()` | Builds the sidebar navigation based on the current `mode`. |
| `renderGridView(data)` | Creates a grid of media cards from an array of items. |
| `renderContentRow(title, items, navKey)` | Renders a horizontal scrolling row of media cards. |
| `renderHero(item)` | Generates the cinematic hero section for the home page. |

### Navigation (`navigation.js`)

Handles all routing and page loading logic. It coordinates between the `State`, `UI`, and external APIs to fetch and display content.

| Function | Description |
|----------|-------------|
| `selectNav(key)` | Navigates to a specific menu section or view. |
| `switchMode(m)` | Changes the browsing mode (TV, Movie, Anime). |
| `handleSearchKey(e)` | Processes search queries and displays results. |
| `loadNextPage()` | Implements infinite scrolling for grid views. |

---

### Data Persistence (`db.js`)

A lightweight wrapper for IndexedDB that handles CRUD operations for user profiles.

| Function | Description |
|----------|-------------|
| `DB.getProfiles()` | Retrieves all profiles. |
| `DB.saveProfile(profile)` | Saves or updates a profile. |
| `DB.deleteProfile(id)` | Deletes a profile by its ID. |

---

### Player (`player.js`)

Manages video playback, including local file streaming, external URLs, and plugin-provided embeds. It handles the UI controls, seeking, and progress tracking.

| Function | Description |
|----------|-------------|
| `renderPlayer()` | Initializes the player UI based on the selected item. |
| `playItem(id, season, episode, path)` | Starts playback. |
| `closePlayer()` | Saves progress, updates achievements, and closes the player. |
| `seekVideo(e)` | Handles click-to-seek on the progress bar. |
| `togglePlay()` | Pauses or resumes the video. |
## Backend Engine (Electron Main Process)

The backend runs in a Node.js environment and is responsible for heavy lifting tasks, security, and hardware interaction.

### Local Media Server (`express`)
A lightweight HTTP server that streams videos with hardware-accelerated transcoding. It automatically detects the user's GPU (NVENC, QuickSync, AMF, VideoToolbox) and falls back to software encoding if necessary.

- **Endpoint:** `http://localhost:8000/stream?path={encodedPath}&time={seekTime}`

### Plugin Sandbox (`vm`)
All community plugins are executed in a Node.js `vm` context. This isolates plugin code, preventing it from crashing the main application or accessing sensitive system resources directly.

**Key Features:**
- **Metadata Injection:** Automatically injects `@name` and `@version` into plugin files.
- **Tripwire System:** Disables plugins that throw errors and logs diagnostic information.
- **State Management:** Plugins are loaded/unloaded dynamically without restarting the app.

### Hardware Acceleration
The `detectHardwareEncoder()` function probes the system for compatible GPUs and configures FFmpeg accordingly. This ensures optimal performance on a wide range of hardware.

| Encoder | OS | Flags |
| :--- | :--- | :--- |
| **NVENC** | Windows/Linux | `h264_nvenc`, `hwaccel cuda` |
| **QuickSync** | Windows/Linux | `h264_qsv`, `hwaccel qsv` |
| **VideoToolbox** | macOS | `h264_videotoolbox` |
| **AMF** | Windows | `h264_amf`, `hwaccel d3d11va` |
| **libx264** | All | Software fallback |

## Plugin Development

Plugins are the heart of Lumière's extensibility. They allow developers to add new content providers, modify the UI, or introduce entirely new features.

### Plugin Lifecycle

1.  **Installation:** A `.js` file is selected via the UI. The backend copies the file into the `userData/plugins` directory.
2.  **Validation:** The plugin is parsed. Missing `@name` or `@version` tags are automatically added.
3.  **Sandboxing:** The code is executed in a Node.js `vm` context.
4.  **Frontend Injection:** If the plugin is active, its code is converted to a Blob URL and injected into the frontend as a `<script>` tag.
5.  **Execution:** Plugin functions (e.g., `fetchCatalog`) are called by the frontend or backend via the `window.api` bridge.
6.  **Error Handling (Tripwire):** If a plugin throws a critical error, it is automatically disabled to prevent crashes.

### Plugin API Reference

A plugin is a standard JavaScript file that exports specific functions depending on the functionality it wants to provide.

#### Metadata Tags
Add these at the top of your plugin file inside a comment block.

```javascript
/**
 * @name My Awesome Source
 * @version 1.0.0
 */
```

#### Exported Functions

| Function | Description | Params | Return |
| :--- | :--- | :--- | :--- |
| `fetchCatalog(endpoint, userAgent, net)` | Fetches a list of media items from your source. | `endpoint`: The API endpoint requested by the frontend.<br>`userAgent`: The Electron user agent string.<br>`net`: The Node.js `net` module for HTTP requests. | `Array<MediaItem>` |
| `getEmbedCode(item, season, episode, subtitleLang)` | Returns an HTML iframe or embed code for a specific media item. | `item`: The media object.<br>`season`: The season number.<br>`episode`: The episode number.<br>`subtitleLang`: The preferred subtitle language. | `string` (HTML iframe code) |
| `getDetails(id)` | (Optional) Fetches detailed metadata (cast, seasons, etc.) for a specific item. | `id`: The ID of the media item. | `Object` (Detailed metadata) |

#### MediaItem Object Structure
When returning items from `fetchCatalog`, the following structure is expected.

```javascript
{
  id: 'unique_id',
  title: 'Movie Title',
  name: 'TV Show Name',
  type: 'movie' || 'tv' || 'anime',
  poster_path: 'https://...', // URL to poster image
  backdrop_path: 'https://...', // URL to backdrop image
  overview: 'Description...',
  release_date: '2023-01-01',
  vote_average: 7.5,
  original_language: 'en'
}
```

### Building a Basic Plugin

Here is a minimal plugin that scrapes a sample API.

```javascript
/**
 * @name Sample Scraper
 * @version 1.0.0
 */

// Plugin must expose a fetchCatalog function
module.exports.fetchCatalog = async function(endpoint, userAgent, net) {
    // The frontend sends a request like: ?api=discover&mode=tv
    // Parse the endpoint to determine what to fetch.
    const url = new URL(`https://my-api.com${endpoint}`);
    const response = await net.fetch(url.toString(), {
        headers: { 'User-Agent': userAgent }
    });

    if (!response.ok) return [];
    const data = await response.json();

    // Transform the data to the required MediaItem format.
    return data.results.map(item => ({
        id: item.id,
        title: item.title || item.name,
        type: item.media_type,
        poster_path: `https://image.tmdb.org/t/p/w500${item.poster_path}`,
        backdrop_path: `https://image.tmdb.org/t/p/original${item.backdrop_path}`,
        overview: item.overview,
        release_date: item.release_date || item.first_air_date
    }));
};

// For player embeds:
module.exports.getEmbedCode = function(item, season, episode, subtitleLang) {
    // Return an iframe string.
    return `<iframe src="https://my-player.com/embed/${item.id}?s=${season}&e=${episode}&lang=${subtitleLang}"></iframe>`;
};
```

## Profiles Engine Documentation

The Profiles Engine (`profile.js`) adds sophisticated user management, parental controls, and authentication.

### Key Features
- **Password Protection:** Profiles can be secured with a PIN.
- **Parental Controls:** Block content based on rating categories (G, PG, PG-13, M, R, X).
- **Manual Ratings:** Users can override the rating for any specific title.
- **Spatial Navigation:** Traps keyboard navigation within modals for accessibility.

### Core Methods

| Method | Description |
| :--- | :--- |
| `filterMedia(mediaArray, profile)` | Filters media based on the profile's blocked ratings and manual overrides. |
| `showPasswordPrompt(profile, onSuccess)` | Displays a cinematic modal asking for the user's PIN. |
| `injectGearIcon()` | Adds a settings gear icon to the profile screen. |
| `openSettingsDashboard()` | Opens the master settings dashboard for managing profiles. |
| `showRatingPrompt(item, currentRating, onSave)` | Allows users to manually set a rating for a specific item. |

### Parental Control Logic
1.  **Check Manual Rating:** If `profile.manualRatings[item.id]` exists, use that rating. If it's `'BLOCK'`, hide the item.
2.  **Map TMDB Rating:** If the item has a `certification` or `adult` flag, map it to the Lumière standardized rating.
3.  **Check Block List:** If the resulting rating is in `profile.blockedRatings`, hide the item.

## Achievements Engine Documentation

The Achievements Engine (`achievements.js`) gamifies the viewing experience by tracking user behavior and unlocking trophies.

### Key Features
- **Event-Driven:** Achievements are unlocked in real-time as the user interacts with the app.
- **Time-Based Tracking:** Tracks viewing streaks, total hours, and session lengths.
- **Personalized Cards:** Generates premium, shareable achievement cards using HTML Canvas.
- **Extensible Database:** The achievement list is easy to add to.

### Achievement Evaluation
The `evaluate(profile, item, season, episode, durationMs)` function is the central dispatcher. It checks for specific conditions based on the current viewing event.

**Examples:**
- **Night Owl:** Watched between 12 AM and 4 AM.
- **Binger:** Watched 8+ episodes of a show in a 12-hour window.
- **Committed:** Accumulated 15+ viewing hours over 3+ days.

### Rendering & Export
Achievements are rendered in a grid view. Clicking on a media card opens a modal showing all unlocked achievements for that specific title. The `exportCard(achId, mediaId)` function generates a high-resolution PNG image using Canvas, complete with dynamic backgrounds, text overlays, and glitch effects.

## Complete Function Reference Table

This table provides a comprehensive overview of all major functions exposed globally or via the API bridge.

### Frontend (UI & Application Logic)

| Function | File | Description |
| :--- | :--- | :--- |
| `loadProfilesScreen()` | `main.js` | Displays the profile selection screen. |
| `selectProfile(id)` | `main.js` | Logs into a profile and loads the main app. |
| `selectItem(id)` | `main.js` | Selects a media item and opens the detail modal. |
| `markWatched(item, season, episode)` | `main.js` | Saves viewing progress and triggers achievements. |
| `renderModal()` | `main.js` | Renders the media detail modal with play buttons and episode lists. |
| `toggleFavorite(id)` | `main.js` | Adds or removes an item from the user's favorites list. |
| `renderNav()` | `ui.js` | Builds the sidebar navigation. |
| `renderMain()` | `ui.js` | Renders the main content area based on the active key. |
| `renderGridView(data)` | `ui.js` | Renders a grid of media items. |
| `selectNav(key)` | `navigation.js` | Navigates to a specific menu section. |
| `switchMode(m)` | `navigation.js` | Switches between TV, Movie, and Anime modes. |
| `handleSearchKey(e)` | `navigation.js` | Handles the search input and displays results. |
| `renderPlayer()` | `player.js` | Renders the video player interface. |
| `playItem(id, season, episode, path)` | `player.js` | Starts playback for a given media item. |
| `closePlayer()` | `player.js` | Stops playback and saves progress. |
| `seekVideo(e)` | `player.js` | Seeks to a specific position in the video. |
| `togglePlay()` | `player.js` | Pauses or resumes playback. |

### Backend (Electron & IPC)

| Function | File | Description |
| :--- | :--- | :--- |
| `createWindow()` | `main_process.js` | Creates the main Electron BrowserWindow. |
| `loadPlugins()` | `main_process.js` | Scans the plugins directory and loads active plugins. |
| `detectHardwareEncoder()` | `main_process.js` | Detects the GPU and configures FFmpeg. |
| `startLocalMediaServer()` | `main_process.js` | Starts the Express server for local streaming. |

### IPC Handles (Preload -> Main)

| Handle | Description |
| :--- | :--- |
| `api.selectMediaFiles()` | Opens a file dialog to select local media. |
| `api.moveMediaFile()` | Moves a file into the Lumière pipeline directory. |
| `api.syncLocalLibrary()` | Scans the pipeline directory for new files. |
| `api.fetchYTS(endpoint)` | Proxies requests to the YTS API. |
| `api.cacheImage(id, url)` | Caches network logos locally. |
| `api.installPlugin()` | Installs a new community plugin. |
| `api.getPlugins()` | Retrieves the list of installed plugins. |
| `api.fetchCatalog(endpoint)` | Calls the `fetchCatalog` function on all active plugins. |
| `api.getPlayerCode(item, season, episode, lang)` | Retrieves the embed code for a media item from plugins. |
| `api.startWatchParty(url, bounds)` | Launches a watch party view. |
| `api.killTranscoder()` | Stops the active FFmpeg process. |
| `api.getVideoDuration(path)` | Gets the duration of a video file. |

### Core Engines

| Engine | File | Description |
| :--- | :--- | :--- |
| `ProfilesEngine` | `profile.js` | Manages profiles, passwords, and parental controls. |
| `AchievementsCore` | `achievements.js` | Manages the achievement system. |
| `AudioEngine` | `audio-engine.js` | Manages audio processing (Night Mode, Volume). |
| `LocalMediaEngine` | `library.js` | Handles local media scanning and import. |
| `DB` | `db.js` | Wrapper for IndexedDB profile operations. |

## Contribution Guide

Thank you for your interest in contributing to Lumière!

### Getting Started
1.  **Fork the Repository:** Create a fork of the main Lumière repo.
2.  **Clone Locally:** `git clone git@github.com:yourname/lumiere.git`
3.  **Install Dependencies:** Navigate to the project root and run `npm install`. This will install all necessary backend dependencies.
4.  **Start the App:** Run `npm start` to launch the Electron app.

### Code Style
- **JavaScript:** Use modern ES6+ syntax. Prefer `const` and `let` over `var`.
- **Comments:** Document all public functions and complex logic using JSDoc.
- **UI:** Use Tailwind CSS for styling. Avoid inline styles.

### Testing Plugins
1.  Place your plugin `.js` file in the `userData/plugins` directory.
2.  Launch the app and navigate to the "Extensions" section in the sidebar.
3.  Your plugin should appear in the list. If it fails, check the console for errors.

### Reporting Bugs
When reporting a bug, please include:
- **OS:** Windows, macOS, or Linux.
- **Version:** The Lumière version (found in the sidebar).
- **Steps to Reproduce:** A clear, concise sequence of actions.
- **Expected vs. Actual:** What you expected to happen versus what actually happened.
- **Logs:** Any relevant error messages from the dev console.

### Pull Request Process
1.  **Create a Branch:** Create a new branch for your feature or bugfix.
2.  **Write Code:** Implement your changes, following the code style.
3.  **Test:** Thoroughly test your changes on multiple platforms if possible.
4.  **Document:** Update the `README.md` or this documentation if necessary.
5.  **Submit:** Open a pull request against the `main` branch.

Thank you for helping make Lumière a better platform for everyone!`;
