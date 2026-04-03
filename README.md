# MOLTamp Widgets

Community widgets for [MOLTamp](https://moltamp.com) — the skinnable shell for Claude Code.

Widgets are self-contained HTML pages that run inside MOLTamp's panels. They can display data, respond to Claude's state, and match any skin's look automatically.

## Install a Widget

1. Download a widget `.zip` from this repo
2. In MOLTamp, open **Settings > Tabs > Import Widget...**
3. Select the `.zip` — done. It appears in your widget picker.

**Or manually:** unzip to `~/Moltamp/widgets/<category>/<widget-name>/` and restart.

## Browse Widgets

| Widget | Category | Description |
|--------|----------|-------------|
| [System Stats](widgets/system-stats/) | System | Live CPU, memory, activity level, and shell state |

*Submit yours via PR.*

---

## Build Your Own Widget

A widget is a folder with two files:

```
my-widget/
  widget.json       <- manifest
  index.html        <- your widget
  assets/           <- optional images, fonts
```

### widget.json

```json
{
  "id": "my-widget",
  "name": "My Widget",
  "version": "1.0.0",
  "description": "One-line description.",
  "author": "Your Name",
  "category": "Custom"
}
```

**Required fields:**
- `id` — Lowercase, hyphens and underscores only. Must be unique.
- `name` — Display name in the widget picker.

**Optional fields:**
- `category` — Groups your widget in the picker (e.g., "System", "Media", "Fun").
- `version` — Semver string.
- `description` — One-line description.
- `author` — Your name.
- `sizing` — `"normal"` (default) or `"compact"`.
- `agents` — Array of agent types that can use this widget. `["*"]` for all.
- `api` — Declares what SDK features your widget uses:

```json
{
  "api": {
    "ipc": ["system.getStats"],
    "stores": ["telemetry.activity"],
    "keyboard": true,
    "shellState": true,
    "audio": false,
    "localAssets": false
  }
}
```

> **Important:** `keyboard`, `shellState`, etc. must be nested inside the `api` object — not at the top level of widget.json. If `keyboard` is not declared under `api`, keystroke forwarding won't activate and your widget won't receive key events.

### index.html

Your widget is a self-contained HTML page. It runs in a sandboxed iframe. You get full HTML/CSS/JS, the `moltamp` SDK on `window.moltamp`, and the active skin's CSS variables.

**Don't wrap in `<!doctype html>`, `<html>`, or `<body>` tags.** The host iframe provides these. Just write a bare `<div>` + `<style>` + `<script>`, like the example below.

```html
<div id="root"></div>

<style>
  #root {
    padding: 8px;
    color: var(--c-chrome-text);
    font-family: 'SF Mono', monospace;
    font-size: 11px;
  }
  .value { color: var(--c-chrome-accent); }
</style>

<script>
  var el = moltamp.el;
  var root = document.getElementById('root');

  var label = el('div', {}, [
    el('span', { color: 'var(--c-chrome-dim)' }, 'Hello '),
    el('span', { color: 'var(--c-chrome-accent)' }, 'World'),
  ]);
  root.appendChild(label);
</script>
```

---

## SDK Reference

The `moltamp` object is available globally. No imports needed. The SDK is auto-injected by the host.

### moltamp.call(channel, ...args) -> Promise

Make an IPC call to MOLTamp. Returns a promise.

```js
var stats = await moltamp.call('system.getStats');
// stats = { cpu: { cores, usagePct, loadAvg1m, loadAvg5m },
//           memory: { totalBytes, freeBytes, usedBytes, usagePct },
//           uptime }
```

**Available channels:**

| Channel | Description | Returns |
|---------|-------------|---------|
| `system.getStats` | CPU and memory stats | `{ cpu, memory, uptime }` |
| `system.weather` | Weather (pass location string) | Weather data |
| `music.nowPlaying` | Current track info | Track object |
| `music.playPause` | Toggle playback | — |
| `music.next` | Next track | — |
| `music.previous` | Previous track | — |
| `telemetry.read` | PTY telemetry | Telemetry data |
| `stats.read` | Session statistics | Stats object |

Unlisted channels are rejected. Timeout: 10 seconds.

### moltamp.subscribe(store, selector, callback) -> unsubscribe

Subscribe to live data from MOLTamp's stores. Fires immediately with current value, then on every change.

```js
var unsub = moltamp.subscribe('telemetry', 'activity', function(level) {
  // level = "idle" | "low" | "high"
  indicator.style.color = level === 'high' ? 'var(--t-green)' : 'var(--c-chrome-dim)';
});
```

**Available stores:**

| Store | Selectors | Description |
|-------|-----------|-------------|
| `telemetry` | `activity`, `bytesPerSec`, `peakBytesPerSec`, `bytesIn`, `bytesOut`, `uptime`, `rateHistory` | PTY data flow |
| `claude` | `sessions` | Claude session data (model, context, cost) |
| `agent` | `agents`, `activeCount` | Tracked subagents |
| `git` | `status` | Branch, changed files, commits |
| `event` | `events`, `eventCount` | Hook events |
| `session` | `sessions`, `layout` | Terminal sessions |

### moltamp.settings

Persist widget-specific configuration. Scoped to your widget ID — won't collide with other widgets.

```js
// Save
await moltamp.settings.write({ theme: 'dark', refreshRate: 5000 });

// Load
var config = await moltamp.settings.read();
// config = { theme: 'dark', refreshRate: 5000 }
```

### moltamp.el(tag, styles, children)

DOM helper. Creates an element with inline styles and children.

```js
var row = moltamp.el('div', { display: 'flex', gap: '8px' }, [
  moltamp.el('span', { color: 'var(--c-chrome-dim)' }, 'CPU'),
  moltamp.el('span', { color: 'var(--c-chrome-accent)' }, '42%'),
]);
```

### moltamp.poll(ms, fn) -> cancel

Run a function on an interval. Auto-cleans up when the widget unloads.

```js
moltamp.poll(3000, async function() {
  var stats = await moltamp.call('system.getStats');
  cpuLabel.textContent = stats.cpu.usagePct + '%';
});
```

### moltamp.fmt

Formatting helpers:

```js
moltamp.fmt.number(1234567)     // "1,234,567"
moltamp.fmt.bytes(1048576)      // "1.0 MB"
moltamp.fmt.duration(3661)      // "1h 1m"
moltamp.fmt.pct(0.856)          // "85.6%"
```

---

## Theme Variables

Your widget receives the active skin's CSS variables. Use them so your widget looks right in every skin.

```css
/* Chrome (UI) */
var(--c-chrome-bg)          /* background */
var(--c-chrome-text)        /* primary text */
var(--c-chrome-accent)      /* accent/highlight */
var(--c-chrome-border)      /* borders */
var(--c-chrome-dim)         /* secondary/muted text */

/* Terminal ANSI */
var(--t-foreground)
var(--t-background)
var(--t-red)  var(--t-green)  var(--t-yellow)
var(--t-blue) var(--t-cyan)   var(--t-magenta)
```

Variables update live when the user switches skins.

---

## Canvas and CSS Variables

**This is the #1 gotcha for widget authors.** CSS variables like `var(--c-chrome-accent)` work in stylesheets and inline styles, but **Canvas 2D context does not resolve them**. If you pass `'var(--t-green)'` to `ctx.fillStyle`, it silently fails and nothing renders.

You must resolve variables to computed color values before using them on a canvas:

```js
// Helper — resolve a CSS variable to its computed value
function getColor(varName, fallback) {
  var val = getComputedStyle(document.documentElement)
    .getPropertyValue(varName).trim();
  return val || fallback;
}

// WRONG — canvas ignores this, renders nothing
ctx.fillStyle = 'var(--t-green)';

// WRONG — color-mix() is CSS-only, canvas can't parse it
ctx.fillStyle = 'color-mix(in srgb, var(--c-chrome-accent) 68%, white)';

// RIGHT — resolved to an actual color like "#00cc66"
ctx.fillStyle = getColor('--t-green', '#00cc66');
```

Call `getColor()` inside your draw loop (not once at startup) so colors update when the user switches skins.

This applies to **all** canvas operations: `fillStyle`, `strokeStyle`, `shadowColor`, `createLinearGradient` color stops, etc.

---

## Dos and Don'ts

### DO

- **Use theme variables for all colors.** `var(--c-chrome-accent)`, not `#4d9fff`. Your widget must look correct in every skin.
- **Use `moltamp.el()` for dynamic DOM.** It's safer than innerHTML and reads cleaner.
- **Poll sparingly.** `moltamp.poll(3000, fn)` is fine. `moltamp.poll(100, fn)` will eat CPU.
- **Persist user preferences.** Use `moltamp.settings.write()` so config survives restarts.
- **Design for ~200px width.** Widgets share panel space. Keep it compact.
- **Handle missing data gracefully.** IPC calls can fail. Show `--` or `...` as placeholders, not blank space.
- **Test with multiple skins.** Light theme, dark theme, high contrast. If you hardcode colors, it will break.
- **Keep file sizes small.** Max 5MB per file, 20MB total. Widgets are lightweight by design.
- **Use the `body` default styles.** The host injects a sensible base: dark background, monospace font, 11px size, no margin. Build on it.

### DON'T

- **Don't use external URLs.** No CDN links, no Google Fonts, no API calls. The iframe sandbox blocks network access. Everything must be self-contained.
- **Don't use `fetch()` or `XMLHttpRequest`.** Sandboxed. Use `moltamp.call()` for data.
- **Don't try to access `window.parent`.** Sandboxed. Use the SDK's `postMessage` bridge.
- **Don't use `document.write()`.** It will blank your widget. Use `moltamp.el()` or standard DOM APIs.
- **Don't set `overflow: auto` on body.** The host sets `overflow: hidden`. Widgets shouldn't scroll — if your content doesn't fit, make it smaller.
- **Don't use ES modules or `import`.** The iframe doesn't support module scripts. Use plain `<script>` tags with vanilla JS.
- **Don't use `async/await` at the top level.** Wrap async code in an IIFE: `(async function() { ... })();`
- **Don't hardcode pixel sizes for layout.** Use percentages or `flex` so your widget adapts to different panel widths.
- **Don't poll faster than 1 second.** There's no reason to update more often than that, and it wastes resources.
- **Don't forget the `widget.json`.** Without it, MOLTamp won't discover your widget.
- **Don't pass CSS variables to Canvas.** `ctx.fillStyle = 'var(--t-green)'` silently fails. Resolve with `getComputedStyle()` first. See [Canvas and CSS Variables](#canvas-and-css-variables) above.
- **Don't use `color-mix()` on Canvas.** It's a CSS function — canvas can't parse it. Pre-compute blended colors in JS if you need them.
- **Don't wrap your HTML in `<!doctype>`, `<html>`, or `<body>`.** The host iframe provides the document structure. Just write bare `<div>` + `<style>` + `<script>`.
- **Don't put `keyboard: true` at the top level of widget.json.** It must be nested under `"api"` — see [widget.json](#widgetjson) above. Top-level placement is ignored and your widget won't receive key events.

---

## Shell State

Listen for Claude's real-time state changes:

```js
window.addEventListener('message', function(e) {
  if (e.data && e.data.type === 'moltamp:push' && e.data.subscriptionId === 'shell-state') {
    var state = e.data.data;
    // state = "idle" | "thinking" | "streaming" | "tool-use" | "permission" | "error" | "complete"
  }
});
```

---

## Packaging for Distribution

Zip your widget folder:

```bash
cd ~/Moltamp/widgets/System/my-widget
zip -r my-widget.zip .
```

**Critical:** `widget.json` and `index.html` must be at the **root** of the zip — not nested inside a subdirectory. If your zip contains `my-widget/widget.json` instead of just `widget.json`, the import will fail.

```
# WRONG — files nested in a folder
my-widget.zip
  └── my-widget/
        ├── widget.json
        └── index.html

# RIGHT — files at the root
my-widget.zip
  ├── widget.json
  ├── index.html
  └── assets/
```

Assets go in an `assets/` subfolder inside the zip.

Users install by dropping the zip into **Settings > Tabs > Import Widget...** or manually extracting to `~/Moltamp/widgets/<category>/<name>/`.

---

## Contributing

1. Fork this repo
2. Add your widget to `widgets/<your-widget>/`
3. Include a screenshot in your widget's folder
4. Open a PR with a description of what it does

All widgets are reviewed for:
- Valid `widget.json` with unique `id`
- No hardcoded colors (uses theme variables)
- No external network requests
- Reasonable file sizes
- Works across multiple skins

---

## License

MIT
