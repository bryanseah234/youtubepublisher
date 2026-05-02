# PRD: youtubepublisher

## Overview
A JavaScript browser console script that automates two YouTube Studio tasks: (1) bulk-publishing all draft (private/scheduled) videos to Public, and (2) sorting a playlist by numeric order extracted from video titles. Paste into YouTube Studio DevTools console to run. No server, no API key needed — drives the YouTube Studio UI directly via DOM automation.

## Goals
- Mode 1: Find all draft videos in YouTube Studio and set them to Public
- Mode 2: Sort a YouTube playlist by number extracted from video titles
- Run entirely in the browser console (no external dependencies)
- Configurable: mode, visibility, sort key, made-for-kids flag

## Non-Goals
- YouTube Data API integration
- CLI tool
- Batch processing across multiple channels
- Thumbnail or description editing
- Scheduling future publish times

## User Stories
- As a YouTuber with many scheduled/private videos, I want to publish them all to Public at once without clicking each one.
- As a playlist curator, I want to reorder a playlist numerically by episode number in the video titles.

## Tech Stack
- **Language**: JavaScript (browser runtime)
- **Runtime**: YouTube Studio (studio.youtube.com) — Chrome DevTools Console
- **APIs used**: YouTube Studio internal DOM (not YouTube Data API)

## Architecture
Single self-executing IIFE (`(() => { ... })()`):
- Config block at top (user-editable constants)
- `waitForElement(selector)` — polls for DOM element with timeout
- `click(element)` — dispatches click event
- `MODE === 'publish_drafts'` → drives draft publish flow
- `MODE === 'sort_playlist'` → drives drag-to-reorder flow

## Features (detailed)

### Mode: Publish Drafts
- Navigates YouTube Studio drafts list
- For each draft: clicks edit, sets visibility to `VISIBILITY` config value
- Sets `MADE_FOR_KIDS` flag
- Clicks "Save" or "Publish"
- Loops through all drafts with delays

### Mode: Sort Playlist
- Reads playlist video list from YouTube Studio DOM
- Extracts numbers from titles via `/\d+/` regex match
- Sorts using `SORTING_KEY` comparator (numeric, fallback to locale compare)
- Drives drag-and-drop reorder UI to match sorted order

### Config Constants
```js
MODE = 'publish_drafts' // or 'sort_playlist'
DEBUG_MODE = true        // extra console logging
MADE_FOR_KIDS = false
VISIBILITY = 'Public'   // 'Public' / 'Private' / 'Unlisted'
SORTING_KEY = (one, other) => number(one.name) - number(other.name)
```

### Resilient DOM Waiting
- `waitForElement(selector, baseEl, timeoutMs)`: polls every 20ms up to 10s
- `sleep(ms)`: async delay between clicks to allow YouTube Studio UI to respond

## Data / Config
No config files. All settings are constants at the top of the script edited before pasting.

## Deployment / Run
1. Navigate to YouTube Studio (studio.youtube.com)
2. Open DevTools Console (F12)
3. Edit config constants at top of script
4. Paste entire script and press Enter

## Constraints & Notes
- **DOM fragility**: YouTube Studio's internal DOM structure changes frequently — selectors may break after YouTube updates
- **Rate limiting**: too-fast clicks may trigger YouTube's rate limiter; `sleep()` delays help but aren't guaranteed
- **No API auth**: drives UI directly; does not require OAuth or API key — but also can't be headless
- **Mode selection**: `MODE` must be set correctly before pasting — running `publish_drafts` in playlist context does nothing useful
- **Numeric sort**: `SORTING_KEY` assumes video titles contain at least one number; titles without numbers fall back to locale comparison
