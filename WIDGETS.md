# Moltamp Widget Authoring Guide v1.0

The single source of truth for building Moltamp widgets.

Build custom widgets for Moltamp. Drop a folder in, it appears. Delete it, gone.

## Quick Start

```
~/Moltamp/widgets/<category>/<your-widget>/
  widget.json       <- Manifest (required)
  index.html        <- Your widget (required)
  assets/           <- Images, fonts (optional)
```

1. Create a folder in `~/Moltamp/widgets/`. The parent folder name is the **category** (e.g., `System`, `Media`, `Fun`).
2. Add `widget.json` and `index.html`.
3. Restart Moltamp (or switch tabs). Your widget appears in the widget picker.

Copy `_template/` from the project's `widgets/` folder for a working example.

---

## widget.json

```json
{
  "id": "my-widget",
  "name": "My Widget",
  "version": "1.0.0",
  "description": "What it does, in one line.",
  "author": "Your Name",
  "category": "Custom",
  "sizing": "normal",
  "agents": ["*"]
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `id` | yes | Unique ID. Lowercase, hyphens and underscores only. |
| `name` | yes | Display name in the widget picker. |
| `version` | no | Semver. Default `1.0.0`. |
| `description` | no | One-liner for the picker tooltip. |
| `author` | no | Your name. Default `User`. |
| `category` | no | Groups your widget in the picker (e.g., `System`, `Media`, `Fun`). |
| `sizing` | no | `"normal"` (collapsible) or `"full"` (fills panel). Default `normal`. |
| `agents` | no | Which agents this widget works with. `["*"]` = all, `["claude"]` = Claude only. Default `["*"]`. |

### The `api` field

Declares what SDK features your widget uses. **Must be nested under `"api"`** — top-level placement is silently ignored.

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

> **Important:** If `keyboard` is not declared under `api`, keystroke forwarding won't activate and your widget won't receive key events.

---

## index.html

Your widget is a self-contained HTML page rendered in a sandboxed iframe. You get:
- Full HTML/CSS/JS
- The `moltamp` SDK on `window.moltamp` (auto-injected)
- Theme CSS variables matching the active skin

**Don't wrap in `<!doctype html>`, `<html>`, or `<body>` tags.** The host iframe provides these. Just write a bare `<div>` + `<style>` + `<script>`:

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

## Sandbox Environment

Widgets run in a **sandboxed iframe** with `allow-scripts` only. Understanding what's available and what's blocked will save you debugging time.

**Available:**
- Full DOM API (`document.createElement`, `getElementById`, etc.)
- `<canvas>` 2D context
- `requestAnimationFrame`
- `setTimeout`, `setInterval` (but prefer `moltamp.poll()` — it auto-cleans up)
- `addEventListener` for keyboard, mouse, touch, and `message` events
- CSS variables from the active skin (injected by host)
- The `moltamp` SDK object on `window`

**Blocked:**
- `fetch()`, `XMLHttpRequest`, `WebSocket` — no network access
- `window.parent`, `window.top`, `parent.postMessage` — cross-origin blocked; the SDK bridges this for you
- ES modules (`import`, `<script type="module">`) — not supported
- `localStorage`, `sessionStorage` — use `moltamp.settings` instead
- `document.cookie` — sandboxed out
- Clipboard API — sandboxed
- Geolocation, camera, microphone — sandboxed
- Forms submission, popups, navigation — sandboxed

**Available but use with caution:**
- `new Audio()` — only if `api.audio: true` in widget.json
- `console.log()` — works for debugging but users can't see it

> **Testing locally:** If you open `index.html` directly in a browser, `moltamp` will be undefined and your widget will crash. Widgets must be loaded inside Moltamp to work. There is no standalone test harness.

---

## SDK Reference

The `moltamp` object is available globally. No imports needed. The SDK is auto-injected by the host.

### moltamp.call(channel, ...args) -> Promise

Make an IPC call to Moltamp. Returns a promise.

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

Subscribe to live data from Moltamp's stores. Fires immediately with current value, then on every change.

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

- `styles` — object of camelCase CSS properties
- `children` — string, element, or array of either

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

| Method | Example |
|--------|---------|
| `fmt.number(n, decimals)` | `fmt.number(1234567)` -> `"1,234,567"` |
| `fmt.bytes(b)` | `fmt.bytes(1048576)` -> `"1.0 MB"` |
| `fmt.duration(seconds)` | `fmt.duration(3661)` -> `"1h 1m"` |
| `fmt.pct(ratio, decimals)` | `fmt.pct(0.856)` -> `"85.6%"` |

### moltamp.onKeyDown(callback) / moltamp.onKeyUp(callback)

Keyboard event listeners. Only active if `api.keyboard: true` in widget.json.

```js
moltamp.onKeyDown(function(key, code, modifiers) {
  console.log('Key pressed:', key, modifiers);
});
```

### moltamp.meta

Widget metadata set by the host:
- `moltamp.meta.id` — your widget's ID
- `moltamp.meta.name` — your widget's display name
- `moltamp.meta.context` — `"panel"` or `"vibes"`

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

## File Structure

```
~/Moltamp/widgets/
  System/
    my-cpu-monitor/
      widget.json
      index.html
    my-disk-usage/
      widget.json
      index.html
  Fun/
    8ball/
      widget.json
      index.html
      assets/
        ball.png
```

The **category folder** (`System`, `Fun`, etc.) is the grouping shown in the widget picker. Name it whatever you want.

**Skin-bundled widgets** can also live inside a skin at `~/Moltamp/skins/<skin>/widgets/<category>/<widget>/`. These take priority over global widgets with the same ID.

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

## Dos and Don'ts

### DO

- **Use theme variables for all colors.** `var(--c-chrome-accent)`, not `#4d9fff`. Your widget must look correct in every skin.
- **Use `moltamp.el()` for dynamic DOM.** It's safer than innerHTML and reads cleaner.
- **Poll sparingly.** `moltamp.poll(3000, fn)` is fine. `moltamp.poll(100, fn)` is not.
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
- **Don't forget the `widget.json`.** Without it, Moltamp won't discover your widget.
- **Don't pass CSS variables to Canvas.** `ctx.fillStyle = 'var(--t-green)'` silently fails. Resolve with `getComputedStyle()` first. See [Canvas and CSS Variables](#canvas-and-css-variables).
- **Don't use `color-mix()` on Canvas.** It's a CSS function — canvas can't parse it. Pre-compute blended colors in JS if you need them.
- **Don't wrap your HTML in `<!doctype>`, `<html>`, or `<body>`.** The host iframe provides the document structure. Just write bare `<div>` + `<style>` + `<script>`.
- **Don't put `keyboard: true` at the top level of widget.json.** It must be nested under `"api"`. Top-level placement is ignored and your widget won't receive key events.

---

## Common Mistakes

Real-world examples of what breaks and why. If you're using an AI to generate your widget, paste this section as context.

### 1. CSS variables in Canvas (silent render failure)

Canvas 2D context can't parse CSS variables or functions. Nothing renders — no error, just invisible output.

```js
// BROKEN — ctx silently ignores the value
ctx.fillStyle = 'var(--t-green)';
ctx.fillRect(0, 0, 10, 10);  // draws nothing

// BROKEN — color-mix() is CSS-only
ctx.fillStyle = 'color-mix(in srgb, var(--c-chrome-accent) 68%, white)';

// FIXED — resolve to a computed value first
function getColor(varName, fallback) {
  return getComputedStyle(document.documentElement)
    .getPropertyValue(varName).trim() || fallback;
}
ctx.fillStyle = getColor('--t-green', '#00cc66');
ctx.fillRect(0, 0, 10, 10);  // draws green
```

### 2. widget.json fields in wrong location (features silently disabled)

```json
// BROKEN — keyboard at top level, widget never receives key events
{
  "id": "my-game",
  "name": "My Game",
  "keyboard": true
}

// FIXED — keyboard nested under api
{
  "id": "my-game",
  "name": "My Game",
  "api": {
    "keyboard": true
  }
}
```

### 3. Full HTML document wrapper (style conflicts)

```html
<!-- BROKEN — host already provides <html>, <head>, <body> with base styles.
     Adding your own can override the SDK injection and break theming. -->
<!doctype html>
<html>
<head><title>My Widget</title></head>
<body>
  <div id="root"></div>
  <script>/* ... */</script>
</body>
</html>

<!-- FIXED — bare fragment, host wraps it for you -->
<div id="root"></div>
<style>/* ... */</style>
<script>/* ... */</script>
```

### 4. Top-level async/await (syntax error in sandbox)

```js
// BROKEN — top-level await not supported
var config = await moltamp.settings.read();

// FIXED — wrap in an IIFE
(async function() {
  var config = await moltamp.settings.read();
  // ... use config
})();
```

### 5. Using fetch/XHR for data (blocked by sandbox)

```js
// BROKEN — network is blocked
var resp = await fetch('https://api.example.com/data');

// FIXED — use the SDK bridge, data comes from the host process
var stats = await moltamp.call('system.getStats');
```

### 6. Using setInterval instead of moltamp.poll (memory leak)

```js
// WORKS but leaks — interval keeps running after widget unloads
setInterval(function() { update(); }, 3000);

// BETTER — auto-cleans up when widget is removed
moltamp.poll(3000, function() { update(); });
```

### 7. Zip with nested directory (import fails)

```bash
# BROKEN — zip contains a folder wrapper
cd ~/widgets
zip -r my-widget.zip my-widget/
# Result: my-widget.zip → my-widget/widget.json (WRONG)

# FIXED — zip from inside the widget folder
cd ~/widgets/my-widget
zip -r my-widget.zip .
# Result: my-widget.zip → widget.json (RIGHT)
```

### 8. Hardcoded colors (breaks on skin switch)

```js
// BROKEN — looks fine in one skin, invisible or ugly in others
ctx.fillStyle = '#4d9fff';
el.style.color = 'white';

// FIXED — adapts to every skin automatically
ctx.fillStyle = getColor('--c-chrome-accent', '#4d9fff');
el.style.color = 'var(--c-chrome-text)';
```

### 9. Using localStorage (not available)

```js
// BROKEN — sandboxed out, throws or returns null
localStorage.setItem('highScore', score);

// FIXED — persists to host filesystem, scoped to your widget ID
await moltamp.settings.write({ highScore: score });
```

### 10. Assuming moltamp.settings.read() returns data on first run

```js
// BROKEN — config is null on first ever load
var config = await moltamp.settings.read();
var rate = config.refreshRate;  // TypeError: cannot read property of null

// FIXED — always provide defaults
var config = (await moltamp.settings.read()) || {};
var rate = config.refreshRate || 3000;
```

---

## Checklist

Before sharing your widget:

- [ ] `widget.json` exists with a unique `id` (lowercase, hyphens/underscores only)
- [ ] `widget.json` has `name` field
- [ ] `api` features (`keyboard`, `shellState`, etc.) are nested under `"api"`, not top-level
- [ ] `index.html` has no `<!doctype>`, `<html>`, `<head>`, or `<body>` tags
- [ ] No `import` statements or `<script type="module">`
- [ ] No top-level `await` — all async code is in an IIFE
- [ ] No `fetch()`, `XMLHttpRequest`, or external URLs
- [ ] No `localStorage` / `sessionStorage` — uses `moltamp.settings` instead
- [ ] No hardcoded colors — all colors use `var(--c-*)` or `var(--t-*)` theme variables
- [ ] If using `<canvas>`: all `fillStyle`/`strokeStyle` values are resolved via `getComputedStyle()`, never raw `var()` strings
- [ ] No `color-mix()` or other CSS functions passed to canvas context
- [ ] `moltamp.settings.read()` handles `null` return on first run
- [ ] Uses `moltamp.poll()` instead of raw `setInterval` for recurring work
- [ ] Zip contains `widget.json` at the root, not inside a subdirectory
- [ ] Total file size under 20MB, individual files under 5MB
- [ ] Tested with at least 2 different skins (one light, one dark)
