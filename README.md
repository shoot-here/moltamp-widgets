<div align="center">

<img src="https://raw.githubusercontent.com/shoot-here/moltamp-widgets/main/.github/assets/banner.svg" alt="Moltamp Widgets — sandboxed mini-apps for AI terminals" width="100%"/>

<br/><br/>

# Moltamp Widgets

**Sandboxed HTML mini-apps for the Moltamp cockpit shell — clocks, system monitors, audio visualizers, mini games, and more for Claude Code, Codex CLI, Gemini CLI, and Aider.**

[![License: MIT](https://img.shields.io/github/license/shoot-here/moltamp-widgets?style=flat-square&color=4d9fff&labelColor=08080a)](LICENSE)
[![Widgets](https://img.shields.io/badge/widgets-1-22c55e?style=flat-square&labelColor=08080a)](#featured-widgets)
[![Spec](https://img.shields.io/badge/spec-WIDGETS.md-4d9fff?style=flat-square&labelColor=08080a)](WIDGETS.md)
[![Gallery](https://img.shields.io/badge/gallery-moltamp.com-ff6b4d?style=flat-square&labelColor=08080a)](https://moltamp.com/community)

**[Download Moltamp](https://moltamp.com)** &nbsp;&middot;&nbsp; **[Browse the Gallery](https://moltamp.com/community)** &nbsp;&middot;&nbsp; **[Widget Spec](WIDGETS.md)** &nbsp;&middot;&nbsp; **[Contributing](CONTRIBUTING.md)**

</div>

<br/>

## What's this repo?

This is the open-source widget library for **[Moltamp](https://moltamp.com)** — a skinnable cockpit UI for AI coding terminals like **Claude Code**, **Codex CLI**, **Gemini CLI**, and **Aider**. Widgets are bite-sized HTML pages that live in Moltamp's side panels and vibes banner — clocks, system monitors, weather, notes, audio visualizers, mini games, anything you can build with HTML, CSS, and vanilla JS.

Every widget runs in a **sandboxed iframe** with no network access. Data flows in through a typed SDK bridge (`moltamp.call()`, `moltamp.subscribe()`, `moltamp.poll()`), and colors come from the active skin's CSS variables — so your widget automatically retones for every theme without a single line of extra code.

If you want a Claude Code companion that feels alive — vibe coding alongside a CPU graph, a lo-fi visualizer, and a Pomodoro timer — this is the place.

> **Keywords:** Claude Code widgets, AI terminal widgets, terminal dashboard, Moltamp widgets, vibe coding, system monitor widget, Codex CLI widgets, Gemini CLI widgets, Aider widgets, developer tools.

<br/>

## Featured Widgets

| Widget | Category | What it does |
|--------|----------|-------------|
| [System Stats](widgets/system-stats/) | System | Live CPU, memory, activity level, and shell state |
| *Your widget here* | — | [Submit a PR](#contribute-a-widget) |

Browse every community widget (with live previews) at **[moltamp.com/community](https://moltamp.com/community)**.

<br/>

## What's in this repo

```
moltamp-widgets/
├── widgets/            <- One folder per widget (drop-in installable)
│   └── system-stats/
│       ├── widget.json
│       └── index.html
├── WIDGETS.md          <- Full SDK + manifest spec — read this first
├── CONTRIBUTING.md     <- PR checklist + review criteria
└── README.md
```

<br/>

## Install a widget

**Inside Moltamp** (recommended):

> Settings &rarr; Tabs &rarr; Import Widget &rarr; pick the widget folder or `.zip`.

**From the command line:**

```bash
git clone https://github.com/shoot-here/moltamp-widgets.git
cp -r moltamp-widgets/widgets/system-stats ~/.moltamp/widgets/
```

Restart Moltamp (or switch tabs) and the widget appears in your widget picker.

<br/>

## Build a widget in 60 seconds

A widget is two files in a folder:

```
my-widget/
  widget.json       <- manifest
  index.html        <- bare HTML fragment (no <!doctype> wrapper)
  assets/           <- optional: images, fonts, sounds
```

**`widget.json`**

```json
{
  "id": "my-widget",
  "name": "My Widget",
  "version": "1.0.0",
  "description": "What it does, in one line.",
  "author": "Your Name",
  "category": "Custom",
  "api": {
    "ipc": ["system.getStats"],
    "stores": ["telemetry.activity"]
  }
}
```

**`index.html`**

```html
<div id="root"></div>

<style>
  #root { padding: 8px; color: var(--c-chrome-text); font: 11px monospace; }
  .value { color: var(--c-chrome-accent); }
</style>

<script>
  (async function() {
    var root = document.getElementById('root');
    moltamp.poll(3000, async function() {
      var stats = await moltamp.call('system.getStats');
      root.innerHTML = '';
      root.appendChild(moltamp.el('div', {}, [
        moltamp.el('span', { color: 'var(--c-chrome-dim)' }, 'CPU '),
        moltamp.el('span', { color: 'var(--c-chrome-accent)' }, stats.cpu.usagePct + '%')
      ]));
    });
  })();
</script>
```

That's a working widget. The full SDK reference, available IPC channels, store subscriptions, lifecycle hooks, and Canvas tips all live in **[WIDGETS.md](WIDGETS.md)**.

<br/>

## Generate one with AI

Point Claude, ChatGPT, Codex, or any LLM at **[WIDGETS.md](WIDGETS.md)** — it includes a *"Common Mistakes"* section with the top 10 gotchas AI tools hit, plus complete code examples for each. The sandbox catches most dangerous mistakes (network calls, cross-origin access, ES module imports), so AI-generated widgets are safe by design.

<br/>

## Contribute a widget

1. **Fork** this repo
2. Create `widgets/your-widget-id/` with `widget.json` + `index.html`
3. Add a `preview.png` screenshot
4. Run through the checklist in **[WIDGETS.md](WIDGETS.md#checklist)** and **[CONTRIBUTING.md](CONTRIBUTING.md)**
5. **[Open a PR](../../pulls)** — we review weekly

Merged widgets ship in the next Moltamp release and appear in the in-app picker plus **[moltamp.com/community](https://moltamp.com/community)**.

<br/>

## License

[MIT](LICENSE) — use, fork, remix, ship. Attribution appreciated but not required.

<br/>

<div align="center">

<a href="https://moltamp.com">
  <img src=".github/assets/logo.png" alt="Moltamp" width="32"/>
</a>

<br/>

<sub>Made for the community by <a href="https://moltamp.com">Moltamp</a></sub>

</div>

<!-- repo topics: moltamp, claude-code, claude-code-widgets, ai-terminal, terminal-widgets, electron, vibe-coding, sandbox, codex-cli, gemini-cli, aider, developer-tools, html-widgets, dashboard -->
