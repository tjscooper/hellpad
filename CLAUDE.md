# CLAUDE.md — HELLPAD Project Context

## What this project is

A Helldivers 2 cosplay prop for Tim's son. It's a wrist-mounted forearm display that lets him enter Helldivers stratagem codes (directional sequences: Up/Down/Left/Right) to "call in" reinforcements, weapons, and airstrikes. Runs as a PWA served from GitHub Pages so his son can load it on his phone.

## Key decisions & preferences

- **Single-file app** — everything in `index.html`. No build tools, no framework, no npm. Keep it that way.
- **No summary at end of responses** — Tim reads the diff, doesn't need a recap.
- **Landscape by default** — the phone is strapped horizontally to a forearm. All layout decisions should assume landscape orientation.
- **Cosplay-first UX** — the primary flow is: enter a code → see a dramatic flash overlay confirming the stratagem. Everything else (list, home screen) is secondary.
- **Push when asked** — Tim will explicitly say "go ahead and push" or similar. Don't push speculatively.

## Repo & deployment

- GitHub remote: `https://github.com/tjscooper/hellpad.git`
- Branch: `main`
- Live URL: `https://tjscooper.github.io/hellpad/`
- Deploy: GitHub Actions on push to `main` (`.github/workflows/deploy.yml`). Pages source must be set to "GitHub Actions" in repo Settings → Pages.
- No build step — static files only.

## Architecture

Single file: `index.html` (~700 lines). See `README.md` for full DOM/JS reference.

**Two screens:**
1. `#home-screen` — boot screen overlay (z-index 1000), shown on load
2. `#app` — keypad (starts `display:none`)

**Navigation FABs (fixed position, always visible):**
- `#home-fab` bottom-left — `▶` on home, `⌂` on keypad
- `#fab` bottom-right — list toggle, hidden on home screen

**State:** `inp` array of `"U"/"D"/"L"/"R"` strings. `onHome` bool.

## Stratagem data

Hardcoded in `index.html` as `const S = [...]`. Each entry: `{n, c, k}` (name, category, code array).
Icon CDN: `https://cdn.jsdelivr.net/gh/nvigneux/Helldivers-2-Stratagems-icons-svg@master/{folder}/{file}.svg`
Icon mapping: `const ICONS = {...}` object in the same file.

## Visual style

- Dark blue/cyan "holographic wrist computer" theme for the keypad
- Home screen: bright blue radial gradient (`#4ab8e8` center → `#041028` outer), skull+wings emblem, large bold italic title, segmented progress bar
- D-pad: white SVG polygon arrows on blue radial gradient panel, inverted-T keyboard layout
- Flash overlay: full-screen dark overlay with 240px stratagem icon, name, code, 4s countdown bar
- Scanline overlay: `body::after` repeating-linear-gradient

## Timings (as of last update)

- Flash visible: 4000ms
- Flash bar animation: 4000ms
- Auto-clear after match: 4800ms
- Boot animation: ~2500ms

## Icon source

`https://github.com/nvigneux/Helldivers-2-Stratagems-icons-svg` served via jsDelivr CDN. CORS-safe for GitHub Pages. Falls back to emoji if image fails to load. Folder structure matches in-game ship module names (e.g. `Patriotic Administration Center/`, `Orbital Cannons/`, `Hangar/`).
