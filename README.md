<div align="center">

<a href="https://moltamp.com">
  <img src=".github/assets/hero-link.png" alt="MOLTamp — Skinnable Shell for AI Terminals" width="720" style="border-radius: 12px; margin: 16px 0;"/>
</a>

<br/><br/>

[![License: MIT](https://img.shields.io/badge/license-MIT-4d9fff.svg?style=flat-square&labelColor=08080a)](LICENSE)
[![Widgets](https://img.shields.io/badge/widgets-1-22c55e.svg?style=flat-square&labelColor=08080a)](#browse-widgets)
[![Spec](https://img.shields.io/badge/spec-v1.0-4d9fff.svg?style=flat-square&labelColor=08080a)](WIDGETS.md)
[![Website](https://img.shields.io/badge/moltamp.com-widgets-ff6b4d.svg?style=flat-square&labelColor=08080a)](https://moltamp.com/widgets/)

**[Download MOLTamp](https://moltamp.com)** &nbsp;&middot;&nbsp; **[Widget Guide](WIDGETS.md)** &nbsp;&middot;&nbsp; **[Contributing](CONTRIBUTING.md)**

</div>

<br/>

## What Are Widgets?

MOLTamp wraps Claude Code's terminal in a skinnable cockpit UI — vibes panel, side panels, telemetry ticker, reactive animations. **Widgets** are self-contained HTML pages that run inside MOLTamp's panels. They display live data, respond to Claude's state, and automatically match any skin's color palette. Think of them as mini-apps living alongside your terminal.

> Self-contained HTML. Full SDK access. Skin-aware by default.

Widgets can show system stats, clocks, weather, notes, audio visualizers, games — anything you can build with HTML, CSS, and vanilla JS.

<br/>

## Browse Widgets

| Widget | Category | Description |
|--------|----------|-------------|
| [System Stats](widgets/system-stats/) | System | Live CPU, memory, activity level, and shell state |
| *Your widget here* | | [Submit a PR](#contributing) |

---

## Install a Widget

**From MOLTamp UI:**
1. Download a widget `.zip` from this repo (or the [marketplace](https://moltamp.com))
2. In MOLTamp, open **Settings > Tabs > Import Widget...**
3. Select the `.zip` — done. It appears in your widget picker.

**Manual install:**
```bash
git clone https://github.com/shoot-here/moltamp-widgets.git
cp -r moltamp-widgets/widgets/system-stats ~/Moltamp/widgets/System/
```

Restart MOLTamp (or switch tabs) and your widget appears.

<br/>

## Build Your Own Widget

A widget is a folder with two files:

```
my-widget/
  widget.json       <- manifest (id + name required)
  index.html        <- bare HTML fragment (no <!doctype> wrapper)
  assets/           <- optional images, fonts, sounds
```

### widget.json

```json
{
  "id": "my-widget",
  "name": "My Widget",
  "version": "1.0.0",
  "description": "What it does, in one line.",
  "author": "Your Name",
  "category": "Custom",
  "sizing": "normal",
  "agent": "*",
  "api": {
    "ipc": ["system.getStats"],
    "stores": ["telemetry.activity"],
    "keyboard": false,
    "shellState": true,
    "audio": false,
    "localAssets": false
  }
}
```

### index.html

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
  (async function() {
    var root = document.getElementById('root');

    moltamp.poll(3000, async function() {
      var stats = await moltamp.call('system.getStats');
      root.innerHTML = '';
      root.appendChild(
        moltamp.el('div', {}, [
          moltamp.el('span', { color: 'var(--c-chrome-dim)' }, 'CPU '),
          moltamp.el('span', { color: 'var(--c-chrome-accent)' }, stats.cpu.usagePct + '%'),
        ])
      );
    });
  })();
</script>
```

> **Full spec:** [WIDGETS.md](WIDGETS.md) — SDK reference, IPC channels, store subscriptions, lifecycle, Canvas gotchas, and more.

---

## Key Concepts

**Sandboxed iframe** — No network, no localStorage, no ES modules. Data comes from the `moltamp` SDK bridge.

**Theme variables** — Use `var(--c-chrome-accent)`, `var(--c-chrome-text)`, `var(--t-green)`, etc. Your widget adapts to every skin automatically.

**SDK bridge** — `moltamp.call()` for IPC, `moltamp.subscribe()` for live data, `moltamp.poll()` for intervals, `moltamp.settings` for persistence.

**Canvas gotcha** — Canvas 2D context cannot resolve CSS variables. Use `getComputedStyle()` to resolve them to hex first.

---

## Guidelines

| # | Guideline | Why |
|---|-----------|-----|
| 1 | No `<!doctype>`, `<html>`, or `<body>` wrapper | Host iframe provides these |
| 2 | No `fetch()`, `XMLHttpRequest`, or external URLs | Sandbox blocks network |
| 3 | No `import` or `<script type="module">` | Not supported in sandbox |
| 4 | No top-level `await` — wrap in IIFE | Syntax error in sandbox |
| 5 | All colors via CSS theme variables | Must work in every skin |
| 6 | Use `moltamp.poll()` not `setInterval` | Auto-cleanup on unload |
| 7 | API features nested under `"api"` in widget.json | Top-level is silently ignored |
| 8 | Handle `null` from `moltamp.settings.read()` | First run returns null |
| 9 | Canvas: resolve CSS vars via `getComputedStyle()` | Canvas can't parse `var()` |

---

## Using AI

Point ChatGPT, Claude, or Codex at [WIDGETS.md](WIDGETS.md) — it has a "Common Mistakes" section covering the top 10 gotchas AI tools hit, with code examples for each.

The sandbox catches most dangerous mistakes (network calls, cross-origin access), so AI-generated widgets are safe by design.

---

## Contributing

1. Fork this repo
2. Create `widgets/<your-widget-id>/` with `widget.json` + `index.html`
3. Run through the [checklist](WIDGETS.md#checklist)
4. [Open a PR](../../pulls)

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full guide, review criteria, and templates.

<br/>

<div align="center">

<a href="https://moltamp.com">
  <img src=".github/assets/logo.png" alt="MOLTamp" width="32"/>
</a>

<br/>

<sub>Made for the community by <a href="https://moltamp.com">MOLTamp</a></sub>

</div>
