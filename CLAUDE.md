# CLAUDE.md — HarkenJot

## Project Overview

HarkenJot is a **single-file web application** for taking notes while consuming audio, video, and text content. The entire application lives in `HarkenJot.html` (~6,000 lines). There is no build system, no package manager, and no backend server.

## Architecture

### Single-File Design

Everything — HTML, CSS, and JavaScript (React/JSX) — is in one file: `HarkenJot.html`. Dependencies are loaded from CDNs at runtime. Babel Standalone transpiles JSX in the browser.

### File Structure

```
HarkenJot.html   # The entire application
README.md        # Project documentation
CLAUDE.md        # This file
```

### Key Sections in HarkenJot.html

| Line Range | Section |
|------------|---------|
| 1–10 | Head, CDN script imports (React 18, Babel, PDF.js) |
| 14–165 | `webSpeechAPI` — Browser-native speech recognition module |
| 67–165 | `whisperASR` — Offline Whisper AI speech recognition fallback |
| 233–1771 | `<style>` — All CSS, including CSS variables for theming |
| 1776–1778 | React hooks imports |
| 1780–1815 | `Icons` — SVG icon components |
| 1817–1838 | Utility functions (`generateId`, `safeHostname`, `formatTime`, etc.) |
| 1840–1903 | `Toast` — Notification component with undo support |
| 1905–1940 | `MediaSessionManager` — Browser Media Session API integration |
| 2015–2038 | localStorage migration (legacy `marginalia_` → `harkenjot_` keys) |
| 2041–2386 | `App` — Root component, state management, tab routing |
| 2388–2463 | `EditableTitle` — Inline title editing component |
| 2465–3921 | `ReaderView` — Article/PDF reader with TTS and voice notes |
| 3923–5557 | `MediaView` — YouTube/podcast player with timestamped notes |
| 5559–5808 | `LibraryView` — Source and note management, import/export |
| 5810–5990 | `NoteSidebar` — Notes display, editing, and navigation |
| ~5995–6004 | `ReactDOM.createRoot` render call |

### Component Hierarchy

```
App
├── Toast
├── ReaderView (forwardRef)
│   └── EditableTitle
├── MediaView (forwardRef)
│   └── EditableTitle
├── LibraryView
└── NoteSidebar
```

### State Management

All state lives in the `App` component via `useState` hooks. There is no external state library. Key state:

- `sources` — Array of source objects (articles, PDFs, videos, podcasts)
- `notes` — Array of note objects linked to sources
- `activeTab` — Current view (`reader`, `media`, `library`)
- `currentSource` — The active source being consumed
- `sidebarOpen` — Notes sidebar visibility
- `carMode` — Simplified large-button UI mode

State is persisted to `localStorage` automatically via `useEffect`:

| localStorage Key | Content |
|-----------------|---------|
| `harkenjot_sources` | All source objects (JSON) |
| `harkenjot_notes` | All note objects (JSON) |
| `harkenjot_positions` | Reading positions in articles/PDFs |
| `harkenjot_media_positions` | Playback positions in media |

### Speech Recognition

Two recognition systems with automatic fallback:

1. **Web Speech API** (`webSpeechAPI`) — Primary. Uses browser-native recognition (Google's service on Chrome/Edge). Requires network.
2. **Whisper AI** (`whisperASR`) — Fallback. Runs `Xenova/whisper-tiny.en` model locally via Transformers.js. Works offline after initial model download.

### External Dependencies (CDN)

| Library | Version | Purpose |
|---------|---------|---------|
| React | 18 | UI framework |
| ReactDOM | 18 | React rendering |
| Babel Standalone | latest | In-browser JSX transpilation |
| PDF.js | 3.11.174 | PDF rendering and text extraction |
| Transformers.js | 2.17.1 | Whisper AI speech recognition (loaded async) |

### External APIs Consumed

- **CORS proxies** — `corsproxy.io`, `api.allorigins.win`, `api.codetabs.com` for fetching articles
- **YouTube IFrame API** — Video playback
- **Spotify oEmbed** — `open.spotify.com/oembed` for podcast metadata
- **iTunes Search** — `itunes.apple.com/search` for podcast discovery
- **RSS feeds** — Custom parser for podcast episodes

### Browser APIs Used

- Web Speech API, Web Audio API (speech recognition)
- localStorage (persistence)
- Media Session API (hardware media controls)
- Screen Wake Lock API (prevent sleep during playback)
- Clipboard API (copy notes)
- Fetch API with CORS proxy fallbacks

## Development Workflow

### Running Locally

```bash
# Option 1: Open directly in browser
open HarkenJot.html

# Option 2: Serve locally (recommended for full CORS support)
python3 -m http.server 8000
# Visit http://localhost:8000/HarkenJot.html
```

### No Build Step

There is no build, compile, or transpile step. Edit `HarkenJot.html` directly and reload the browser. Babel transpiles JSX at runtime.

### No Tests

There is no test suite. Verify changes manually in the browser.

### No Linter or Formatter

There are no linting or formatting tools configured.

## Conventions and Patterns

### Code Style

- **JavaScript**: ES6+ with JSX, transpiled by Babel Standalone in the browser
- **React**: Functional components with hooks (`useState`, `useEffect`, `useRef`, `useCallback`, `useImperativeHandle`)
- **CSS**: All styles in a single `<style>` block using CSS variables (`:root`) for theming
- **IDs**: Generated via `Math.random().toString(36).substr(2, 9)`
- **Naming**: camelCase for functions/variables, PascalCase for React components

### Component Patterns

- `ReaderView` and `MediaView` use `React.forwardRef` with `useImperativeHandle` so `App` can call methods on them (e.g., resuming playback)
- `LibraryView` and `NoteSidebar` are plain function components receiving props
- All inter-component communication is via props (no context or event bus)

### Data Model

**Source object**:
```js
{ id, type, title, url, content, date, ... }
```

**Note object**:
```js
{ id, sourceId, text, position, timestamp, date, ... }
```

Notes reference their parent source via `sourceId`. Position anchoring differs by source type: sentence index for text, timestamp for media.

### Error Handling

- CORS fetch uses a chain of proxy fallbacks
- Speech recognition falls back from Web Speech API to Whisper
- User-facing errors shown via the `Toast` component

## Making Changes

### Adding a New Feature

1. Identify which component section in `HarkenJot.html` to modify
2. Follow existing patterns — functional components, hooks, inline styles via CSS variables
3. Keep everything in the single file (no external modules)
4. Test in Chrome/Edge (primary targets) and verify in Firefox/Safari

### Common Modification Areas

- **UI/Theme**: CSS variables in `:root` (around line 233)
- **Icons**: `Icons` object (line 1780)
- **Reader functionality**: `ReaderView` (line 2465)
- **Media/podcast functionality**: `MediaView` (line 3923)
- **Library/export**: `LibraryView` (line 5559)
- **Notes panel**: `NoteSidebar` (line 5810)
- **App-level state/routing**: `App` (line 2041)

### Version

The current version is shown in the app header. Update it in the `App` component's JSX. **Always increment the version number with every change** using semantic versioning (`MAJOR.MINOR.PATCH`):

- **Patch** (`1.8.x` → `1.8.x+1`) — Small changes and bug fixes
- **Minor** (`1.x.1` → `1.x+1.0`) — New features or functional enhancements
- **Major** (`x.1.1` → `x+1.0.0`) — Only when explicitly requested by the user, or Claude may suggest a major bump for approval when a large portion of the codebase is affected

## Git Conventions

- Branches follow the pattern `claude/<description>-<id>`
- Commit messages are concise and describe the "why" (e.g., "Fix podcast audio missing wake lock on play/pause/end")
- PRs are merged via GitHub merge commits
- **After every push**, always include the GitHub PR creation link in your response text so the in-app "View PR" button works: `https://github.com/jarretmorton/Research-App/pull/new/<branch-name>`

## Deployment

Upload `HarkenJot.html` to any static file host (GitHub Pages, Netlify, Vercel, S3, or any web server). HTTPS is recommended for full browser API support (microphone, Media Session).
