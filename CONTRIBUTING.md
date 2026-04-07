<div align="center">

<a href="https://moltamp.com">
  <img src=".github/assets/logo.png" alt="MOLTamp" width="56"/>
</a>

<br/><br/>

# Contributing to MOLTamp Widgets

**Thanks for building a widget.** Here's how to get it into the repo.

</div>

<br/>

## Quick Version

1. Fork this repo
2. Create `widgets/<Category>/<your-widget-id>/` with `widget.json` and `index.html`
3. Optionally add an `assets/` folder for images or sounds
4. Open a PR using the widget submission template

## Widget Structure

```
widgets/<Category>/<your-widget-id>/
  widget.json       <- Manifest (id + name required)
  index.html        <- Bare HTML fragment (no <!doctype> wrapper)
  assets/           <- Images, sounds (optional)
```

The category folder (e.g., `System`, `Media`, `Fun`, `Productivity`) groups your widget in the picker. Pick the one that fits best, or create a new one.

## Guidelines

The full spec lives in [WIDGETS.md](WIDGETS.md). Here are the rules that matter most for submissions:

1. **Unique widget ID.** Lowercase, hyphens and underscores only. Must not collide with existing widgets.
2. **No external URLs or network calls.** No `fetch()`, no CDN links, no API calls. The sandbox blocks all network access. Use `moltamp.call()` for data.
3. **No ES modules or top-level await.** No `import`, no `<script type="module">`. Wrap async code in an IIFE: `(async function() { ... })();`
4. **Use theme variables for all colors.** `var(--c-chrome-accent)`, not `#4d9fff`. Your widget must look correct in every skin.
5. **Handle null settings on first run.** `moltamp.settings.read()` returns `null` on the first load. Always provide defaults: `var config = (await moltamp.settings.read()) || {};`
6. **Use `moltamp.poll()` not `setInterval`.** Poll auto-cleans up when the widget unloads. `setInterval` leaks.
7. **Design for ~200px width.** Widgets live in narrow side panels. Use percentages and flex, not fixed pixel widths.
8. **API features nested under `"api"` in widget.json.** `keyboard: true` at the top level is silently ignored. It must be `"api": { "keyboard": true }`.

## Testing Checklist

Before submitting, verify:

- [ ] Widget loads without errors in MOLTamp
- [ ] All colors use theme CSS variables (`var(--c-*)` or `var(--t-*)`)
- [ ] Handles missing data gracefully (shows `--` or `...`, not blank space)
- [ ] Tested with at least 2 skins (one light, one dark)
- [ ] No console errors (check with Cmd+Shift+I in MOLTamp)
- [ ] Poll rate >= 1 second (3 seconds recommended)
- [ ] If using Canvas: CSS variables resolved via `getComputedStyle()`, not raw `var()` strings
- [ ] `index.html` has no `<!doctype>`, `<html>`, `<head>`, or `<body>` tags
- [ ] Total file size under 20MB, individual files under 5MB

## AI-Generated Widgets Welcome

Widgets generated with ChatGPT, Claude, Codex, or any other AI tool are accepted and encouraged. The same quality bar applies: your widget must pass all the guidelines above and work across multiple skins.

See the [CLAUDE.md](CLAUDE.md) file for an AI-friendly rule summary you can paste directly into your prompt. MOLTamp's sandbox also prevents most common AI mistakes from causing harm -- but the widget still needs to actually work.

If you used AI, you're welcome to credit it in the `author` field (e.g., `"author": "Your Name + Claude"`), but it's not required.

## Review Criteria

When reviewers evaluate your PR, they check:

| Criteria | What we look for |
|----------|-----------------|
| **Loads without errors** | Widget renders correctly on first load. No console errors. |
| **Theme compliance** | All colors come from CSS variables. Looks good in both light and dark skins. |
| **Missing data handling** | Shows placeholders when IPC calls fail or return null. Never blank. |
| **Reasonable poll rate** | Polls at 1 second minimum, 3 seconds recommended. No 100ms polling. |
| **Sandbox compliance** | No `fetch()`, no `import`, no `localStorage`, no document wrapper. |
| **Visual quality** | Clean layout, readable text, intentional design. Doesn't have to be complex, just polished. |
| **API nesting** | Features like `keyboard`, `audio`, `localAssets` are under `"api"` in widget.json. |

We'll work with you if something needs adjustment. The goal is to get your widget in, not to gatekeep.

## Code of Conduct

Be respectful. Share your creativity. Help others build great widgets.

<br/>

<div align="center">

<sub><a href="https://moltamp.com">moltamp.com</a> &nbsp;&middot;&nbsp; <a href="WIDGETS.md">Authoring Guide</a> &nbsp;&middot;&nbsp; <a href="README.md">Back to README</a></sub>

</div>
