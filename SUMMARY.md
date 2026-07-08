# Heat Index Monitor — Project Summary

## What this is
A mobile-first PWA widget that replaces the manual Excel/photo Heat Index report for **CP S-04 Temfacil**. It pulls live temp/humidity from a Wunderground personal weather station and auto-computes the Heat Index, PAGASA-style danger category, work/rest schedule, and water requirement — no more retyping numbers off the station photo.

## Files (all in one folder, needed together)
- `index.html` — the app UI + logic (single page, no build step, vanilla JS)
- `manifest.json` — PWA manifest (name, icons, standalone display mode)
- `icon.png` — app icon
- `sw.js` — service worker for offline shell caching + installability

## Data source
Wunderground PWS Current Conditions API:
```
https://api.weather.com/v2/pws/observations/current?stationId={ID}&format=json&units=m&apiKey={KEY}
```
- Called directly from the browser (client-side fetch), no backend.
- Response fields used: `observations[0].humidity` and `observations[0].metric.temp`.
- Station ID + API key are entered in the Settings sheet (gear icon) and kept in memory only — not persisted anywhere yet.
- **Known open question:** whether Wunderground's CORS policy allows this direct browser call for Francis's account. Untested with real credentials as of this summary. If it fails with a CORS error (not an auth error), the fix is a small server-side proxy (e.g. a Cloudflare Worker or tiny Node endpoint) that forwards the request.

## Heat Index calculation
Rothfusz regression (NOAA), computed in °F internally then converted back to °C, with the standard low-humidity/high-humidity adjustment terms. Verified against the Excel sample (34.7°C, 61% RH → 44.6°C) — matches exactly.

Category bands (PAGASA-style, matching the original Excel reference table):
| Range | Category | Work/Rest | Water |
|---|---|---|---|
| 27–32°C | Caution | Regular breaks | 1 cup / 20–30 min |
| 33–39°C | Extreme Caution | 50 min work / 10 min rest | 1 cup / 20 min |
| 39–50°C | Danger | 40 min work / 10 min rest | 1 cup / 15 min |
| >50°C | Extreme Danger | Flexible working hours | 1 cup / 10 min |

## Current UI structure
- Hero card: circular progress ring + Heat Index number + category pill + temp/humidity
- Two advisory chips: Work/Rest, Water Requirement
- Status row: connection dot + manual refresh link
- Settings bottom sheet: station ID, API key, auto-refresh toggle (5 min interval), manual temp/humidity entry fallback
- Reference bottom sheet: full category table

## Design tokens (for consistency if extending)
- Background `#0B1220`, panels `#141F30` / `#1B2A40`, borders `#28394F`
- Accent amber `#F0A339`; category colors: caution `#F5C518`, extreme caution `#E8792B`, danger `#D9552B`, extreme danger `#D64545`, safe `#3FA672`
- Fonts: Oswald (display/headers), Inter (body), IBM Plex Mono (data/labels) — currently relying on system fallbacks, real fonts not yet loaded via `<link>` or bundled

## Known gaps / next steps to pick up in Claude Code
1. **Test the live fetch** with real station ID + API key — confirm CORS works or build a proxy.
2. **Persist settings** — station ID/API key currently reset on page reload; consider `localStorage` (works fine outside claude.ai artifacts) so it's not re-entered every time.
3. **Load real webfonts** (Oswald / Inter / IBM Plex Mono via Google Fonts or self-hosted) — currently falls back to system fonts.
4. **Deploy to GitHub Pages** (same pattern as his other PWAs — Waste Management System, HSE Suite) so the manifest + service worker actually enable "Add to Home Screen" install (requires https).
5. **Optional:** true home-screen widget (Android App Widget / iOS Lock Screen widget) — the current build is an installed full-screen PWA, not a native OS widget; that would need a different approach (e.g. a native wrapper or a widget-specific API) if still wanted.
6. **Optional:** background auto-fetch even when the app isn't open (would need a push/background-sync strategy, since browser JS timers stop when the tab isn't active).
