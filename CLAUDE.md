# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running locally

No build step or package manager. Serve the directory with any static file server — the fonts use relative paths so `file://` won't work for them:

```
python -m http.server 8080
# or
npx serve .
```

Then open `http://localhost:8080/Siraphop.html`.

## Project structure

| File | Purpose |
|---|---|
| `Siraphop.html` | Main portfolio page (single-page, anchor-nav) |
| `post-agency-to-inhouse.html` | Blog post template |
| `colors_and_type.css` | Shared design tokens + Kanit @font-face declarations |
| `tweaks-panel.jsx` | Reusable React component library for the live-edit panel |
| `fonts/` | Self-hosted Kanit TTF files (all weights + italics) |

## Architecture

**Design tokens live in `colors_and_type.css`.** All CSS custom properties (`--paper`, `--ink`, `--accent`, `--r-*`, `--shadow-*`, etc.) are declared there. Page-specific layout styles live in inline `<style>` blocks inside each HTML file — never in the shared CSS.

**The tweaks system is a live-edit layer** intended for prototyping and host-side visual tuning, not production theming. Each page declares a `TWEAK_DEFAULTS` JSON literal wrapped in `/*EDITMODE-BEGIN*/` … `/*EDITMODE-END*/` comments. The tweaks panel reads those defaults on load, lets you adjust values in a floating UI, and posts `__edit_mode_set_keys` messages via `window.postMessage` — a host (like a Claude Artifacts iframe) can intercept those messages and rewrite the `EDITMODE` block on disk to persist changes.

**React is CDN-only, used only for the tweaks panel.** The page loads React 18, ReactDOM, and Babel Standalone from unpkg, then loads `tweaks-panel.jsx` as `type="text/babel"` so Babel compiles it in-browser. `tweaks-panel.jsx` exposes every component as a `window` global (`useTweaks`, `TweaksPanel`, `TweakSlider`, etc.). The in-page `<script type="text/babel">` block then uses those globals to mount a `TweaksApp` component that calls `applyTweaks()` whenever tweak values change.

**Nav and scroll behavior** are plain vanilla JS at the bottom of each page — IntersectionObserver for active link state, a scroll listener for the sticky nav shadow.

## Adding a new blog post

1. Copy `post-agency-to-inhouse.html` and rename it.
2. Add a new `<a class="writing-row">` entry in the Writing section of `Siraphop.html` pointing to the new file.
3. The post page shares the same nav/footer markup — keep them in sync manually.

## Accent color

The accent is defined as a static hex in `colors_and_type.css` (`--accent: #FF5C2E`) but overridden at runtime by the tweaks panel using HSL components from `TWEAK_DEFAULTS`. If you change the static hex, also update the matching HSL defaults (`accentHue`, `accentSat`, `accentLight`) in each HTML file's `TWEAK_DEFAULTS`.
