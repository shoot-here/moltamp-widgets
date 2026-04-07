# MOLTamp Widgets -- AI Contributor Guide

Community widget repository for MOLTamp. Widgets are self-contained HTML pages rendered in sandboxed iframes with access to the `moltamp` SDK.

## Repository Structure

```
moltamp-widgets/
├── widgets/            <- Category folders containing widgets
│   └── <Category>/
│       └── <widget-id>/
│           ├── widget.json  <- Manifest (required)
│           ├── index.html   <- Widget code (required)
│           └── assets/      <- Images, sounds (optional)
├── WIDGETS.md          <- Full specification (read this first)
├── CONTRIBUTING.md     <- PR guidelines
└── README.md           <- Overview + widget catalog
```

## Critical Rules -- NEVER Violate These

1. **No `<!doctype>`, `<html>`, `<head>`, or `<body>` tags.** The host iframe provides these. Write bare `<div>` + `<style>` + `<script>`.
2. **No `fetch()`, `XMLHttpRequest`, or external URLs.** The sandbox blocks all network. Use `moltamp.call()` for data.
3. **No ES modules.** No `import`, no `<script type="module">`. Use plain `<script>` with vanilla JS.
4. **No top-level `await`.** Wrap async code in an IIFE: `(async function() { ... })();`
5. **No `localStorage` / `sessionStorage`.** Use `moltamp.settings.read()` / `moltamp.settings.write()`.
6. **Use theme CSS variables for ALL colors.** `var(--c-chrome-accent)`, not `#4d9fff`.
7. **Use `moltamp.poll()` instead of `setInterval`.** Auto-cleans up when widget unloads.
8. **Handle null from `moltamp.settings.read()`** on first run. Always provide defaults.
9. **Canvas cannot resolve CSS variables.** Use `getComputedStyle(document.documentElement).getPropertyValue('--var-name')`.
10. **API features must be nested under `"api"` in widget.json.** `keyboard`, `audio`, `shellState`, `localAssets` at the top level are silently ignored.

## When Generating a New Widget

1. Read `WIDGETS.md` fully -- it's the single source of truth
2. Start from the system-stats example widget
3. Use `moltamp.el()` for DOM building, `moltamp.poll()` for intervals, `moltamp.fmt` for formatting
4. Design for ~200px width (panel context) -- use flexible layouts
5. Handle missing data: show `--` or `...` as placeholders, never blank space
6. Poll at 3000ms minimum. Never faster than 1000ms.
7. Test with multiple skins -- if you hardcode colors, it WILL break

## Common AI Mistakes

- Wrapping HTML in `<!doctype html>` / `<html>` / `<body>` -- host already provides these
- Using `fetch()` or `XMLHttpRequest` -- blocked by sandbox. Use `moltamp.call()`.
- Using `import` or `<script type="module">` -- not supported in sandbox
- Top-level `await` -- syntax error in sandbox. Use IIFE wrapper.
- Using `localStorage` -- sandboxed out. Use `moltamp.settings`.
- Putting `keyboard: true` at top level of widget.json instead of under `"api"`
- Passing CSS variables to Canvas `fillStyle`/`strokeStyle` -- silently fails
- Using `color-mix()` on Canvas -- CSS-only function, canvas can't parse it
- Assuming `moltamp.settings.read()` returns data on first run (it returns `null`)
- Using `setInterval` instead of `moltamp.poll()` (leaks on widget unload)
- Polling faster than 1 second (wastes resources)
- Using `document.write()` -- blanks the widget
- Setting `overflow: auto` on body -- host sets `overflow: hidden`
- Hardcoding pixel widths instead of using percentages or flex

## SDK Quick Reference

```js
// IPC call
var stats = await moltamp.call('system.getStats');

// Subscribe to live data
var unsub = moltamp.subscribe('telemetry', 'activity', function(level) { ... });

// Persist settings (scoped to your widget ID)
await moltamp.settings.write({ key: 'value' });
var config = (await moltamp.settings.read()) || {};

// DOM helper
var el = moltamp.el('div', { color: 'var(--c-chrome-accent)' }, 'Hello');

// Polling (auto-cleanup)
moltamp.poll(3000, async function() { ... });

// Formatting
moltamp.fmt.bytes(1048576)    // "1.0 MB"
moltamp.fmt.duration(3661)    // "1h 1m"
moltamp.fmt.pct(0.856)        // "85.6%"
moltamp.fmt.number(1234567)   // "1,234,567"

// Metadata
moltamp.meta.id       // your widget ID
moltamp.meta.context  // "panel" or "vibes"
```

## widget.json Required Fields

```json
{
  "id": "my-widget",     // lowercase, hyphens/underscores only
  "name": "My Widget"
}
```

Optional: `version`, `description`, `author`, `category`, `sizing` (`"normal"` or `"fill"`), `agent` (`"*"` or `"claude"`).

API features (nest under `"api"`): `ipc`, `stores`, `keyboard`, `shellState`, `audio`, `localAssets`.
