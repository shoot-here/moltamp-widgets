# MOLTamp Widgets -- AI Contributor Guide

Community widget repository for MOLTamp. Widgets are self-contained HTML pages rendered in a sandboxed iframe under a strict CSP injected by the host, with access to the `moltamp` SDK for data.

## Repository Structure

```
moltamp-widgets/
├── widgets/            <- Each subfolder is a complete widget
│   └── <widget-id>/
│       ├── widget.json  <- Manifest (required)
│       ├── index.html   <- Widget code (required)
│       └── assets/      <- Images, sounds (optional)
├── WIDGETS.md          <- Full specification (read this first)
├── CONTRIBUTING.md     <- PR guidelines
└── README.md           <- Overview + widget catalog
```

The `category` is set in each widget's `widget.json` (e.g., `"category": "System"`) — it controls how the widget is grouped in the picker, NOT the directory structure. Widgets sit flat under `widgets/<widget-id>/` regardless of category.

## Critical Rules -- NEVER Violate These

1. **No `<!doctype>`, `<html>`, `<head>`, or `<body>` tags.** The host iframe provides these. Write bare `<div>` + `<style>` + `<script>`.
2. **No `fetch()`, `XMLHttpRequest`, `WebSocket`, or external URLs.** A strict CSP injected by the host blocks all network and all remote resource loading (images, stylesheets, fonts, iframes). The sandbox flag alone does NOT block network -- the CSP does. Use `moltamp.call()` for data.
3. **No `eval`, `new Function`, or string-form `setTimeout`/`setInterval`.** The CSP does not grant `unsafe-eval`. Always pass a real function reference: `setTimeout(fn, ms)`, never `setTimeout("fn()", ms)`.
4. **Only `data:` and `blob:` image sources are allowed.** No remote `<img src>`, no `url('https://...')` in CSS, no remote `@font-face`. Bundle assets locally or generate them on a canvas.
5. **No ES modules.** No `import`, no `<script type="module">`. Use plain `<script>` with vanilla JS.
6. **No top-level `await`.** Wrap async code in an IIFE: `(async function() { ... })();`
7. **No `localStorage` / `sessionStorage`.** Use `moltamp.settings.read()` / `moltamp.settings.write()`.
8. **Use theme CSS variables for ALL colors.** `var(--c-chrome-accent)`, not `#4d9fff`.
9. **Use `moltamp.poll()` instead of `setInterval`.** Auto-cleans up when widget unloads.
10. **Handle null from `moltamp.settings.read()`** on first run. Always provide defaults.
11. **Canvas cannot resolve CSS variables.** Use `getComputedStyle(document.documentElement).getPropertyValue('--var-name')`.
12. **API features must be nested under `"api"` in widget.json.** `keyboard`, `audio`, `shellState`, `localAssets` at the top level are silently ignored.

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
- Using `fetch()` or `XMLHttpRequest` -- blocked by the host CSP. Use `moltamp.call()`.
- Loading remote images, fonts, or stylesheets -- CSP only permits `data:` and `blob:` image sources; everything else is blocked
- Using `eval()`, `new Function()`, or passing a string to `setTimeout`/`setInterval` -- blocked (CSP has no `unsafe-eval`). Always pass a real function.
- Using `import` or `<script type="module">` -- not supported in the widget sandbox
- Top-level `await` -- syntax error in widgets. Use IIFE wrapper.
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
