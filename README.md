# HELLPAD — Personal Hellpad System

A Helldivers 2 stratagem input prop for cosplay, served as a static PWA via GitHub Pages. Designed to be strapped to a forearm in landscape orientation; the user enters directional codes with one hand while the display shows the matched stratagem.

**Live URL:** `https://tjscooper.github.io/hellpad/`

---

## Features

- **Boot home screen** — "PERSONAL HELLPAD SYSTEM" boot sequence with animated segmented progress bar (0→100%), skull-and-wings emblem, corner decorations matching the in-game wrist computer aesthetic
- **Stratagem d-pad** — four large white SVG arrow buttons on a blue radial-gradient panel, laid out in the inverted-T keyboard arrow key style (UP alone on top, LEFT/DOWN/RIGHT below)
- **Live code matching** — as directions are entered, the input strip filters to matching stratagems in real time; an invalid sequence auto-clears immediately
- **Match flash overlay** — full-screen overlay on exact match showing the stratagem icon (240px), name, directional code, and a 4-second countdown progress bar
- **Stratagem icons** — loaded from jsDelivr CDN (`cdn.jsdelivr.net/gh/nvigneux/Helldivers-2-Stratagems-icons-svg@master/`), falls back to emoji if unavailable
- **Stratagem list panel** — slides in from the right, shows all 69 stratagems with icons, codes, and category badges; filters live as codes are entered; tap any stratagem to preview it
- **Audio feedback** — Web Audio API beeps: ascending pitch per input, 3-tone chord on match, sawtooth error on invalid
- **Haptic feedback** — `navigator.vibrate` patterns on input, match, and error
- **Landscape orientation** — manifest sets `"orientation": "landscape"`; d-pad scales to fill the available landscape height
- **Fullscreen PWA** — manifest `"display": "fullscreen"`; works as an Add to Home Screen app on iOS and Android
- **Keyboard support** — arrow keys or WASD + Backspace/Escape/Delete

---

## Stratagem Data

69 stratagems across 9 categories:

| Category | Count |
|---|---|
| Mission | 10 |
| Orbital | 12 |
| Eagle | 7 |
| Support (weapons) | 17 |
| Backpack | 7 |
| Emplacement | 4 |
| Mines | 3 |
| Sentry | 7 |
| Exosuit | 2 |

Data is hardcoded in `index.html` as the `S` array. Each entry: `{n: name, c: category, k: code_array}`.
Icon paths are in the `ICONS` object, mapped by stratagem name to a path within the nvigneux icon repo.

---

## Architecture

Single-file app — everything lives in `index.html`. No build step, no dependencies, no framework.

```
hellpad/
├── index.html        # entire app (HTML + CSS + JS, ~700 lines)
├── manifest.json     # PWA manifest (landscape, fullscreen)
├── README.md
├── CLAUDE.md         # instructions for future AI sessions
└── .github/
    └── workflows/
        └── deploy.yml  # GitHub Actions → GitHub Pages on push to main
```

### Screens

1. **Home screen** (`#home-screen`) — shown on load, fixed overlay z-index 1000; boot animation runs via IIFE `bootAnim()`; tap anywhere or ▶ FAB to launch keypad
2. **Keypad** (`#app`) — starts `display:none`, shown by `showKeypad()`

### Key DOM elements

| ID | Purpose |
|---|---|
| `#home-screen` | Boot/home screen overlay |
| `#app` | Keypad shell (flex column) |
| `#strip` | Top input strip (46px) |
| `#strip-arrows` | Mini SVG arrows for current input |
| `#strip-status` | Match count / stratagem name |
| `#dpad-bg` | Blue radial-gradient d-pad container |
| `#dpad-grid` | 3×2 CSS grid (inverted-T layout) |
| `#flash` | Full-screen match overlay (z-index 10000) |
| `#flash-bar` | Countdown progress bar inside flash |
| `#list-panel` | Slide-in stratagem list (z-index 500) |
| `#fab` | List toggle FAB (bottom-right) |
| `#home-fab` | Screen nav FAB (bottom-left) |

### Key JS state & functions

| Name | Purpose |
|---|---|
| `inp` | `string[]` — current directional input (`"U"`,`"D"`,`"L"`,`"R"`) |
| `resetTimer` | auto-clear timeout handle after a match |
| `onHome` | `bool` — which screen is active |
| `addDir(d)` | append direction, check for match/invalid, render |
| `clr()` | clear input and flash |
| `bksp()` | remove last direction |
| `getFiltered()` | stratagems whose code starts with current input |
| `getExact()` | stratagem whose code exactly equals current input |
| `showFlash(s)` | trigger the full-screen match overlay |
| `render()` | re-render strip, list, and match state |
| `showKeypad()` / `showHome()` | screen navigation |
| `toggleList()` / `closeList()` | list panel open/close |
| `bootAnim()` | IIFE on load — animates home screen progress bar |

### Timings

| Event | Duration |
|---|---|
| Flash overlay visible | 4000ms |
| Flash countdown bar | 4000ms (CSS animation) |
| Auto-clear input after match | 4800ms |
| Boot progress bar | ~2500ms (random increments, 28ms interval) |

### D-pad sizing (responsive)

The d-pad scales to fill landscape height without overflowing width:
```css
width:  calc(min(100vw - 40px, (100dvh - 46px - 28px) * 3/2 + 20px));
height: calc(min(100dvh - 46px - 28px, (100vw - 40px) * 2/3 + 20px));
```

### Icon CDN

```
https://cdn.jsdelivr.net/gh/nvigneux/Helldivers-2-Stratagems-icons-svg@master/{folder}/{name}.svg
```

Folders: `General Stratagems/`, `Orbital Cannons/`, `Bridge/`, `Hangar/`, `Patriotic Administration Center/`, `Engineering Bay/`, `Robotics Workshop/`, `Urban Legends/`. Spaces are `encodeURIComponent`-encoded in the URL.

---

## Deployment

GitHub Actions (`deploy.yml`) deploys to GitHub Pages automatically on every push to `main`. Uses the official `actions/upload-pages-artifact` + `actions/deploy-pages` workflow. No build step — the root directory is uploaded as-is.

To trigger a redeploy: push any commit to `main`, or re-run the workflow from the Actions tab.

---

## Adding / Updating Stratagems

Edit the `S` array in `index.html`:
```js
{n:"Stratagem Name", c:"Category", k:["U","D","L","R", ...]},
```

Add an icon mapping to the `ICONS` object:
```js
"Stratagem Name": "Folder Name/File Name.svg",
```

Valid categories: `Mission`, `Orbital`, `Eagle`, `Support`, `Backpack`, `Emplacement`, `Mines`, `Sentry`, `Exosuit`.
