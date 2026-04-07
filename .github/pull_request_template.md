## Widget Submission

**Widget ID:** `your-widget-id`
**Category:** (e.g., System, Media, Fun, Productivity)

### Description
<!-- What does your widget do? One paragraph. -->

### Checklist

- [ ] `widget.json` has unique `id` and `name`
- [ ] `index.html` has no `<!doctype>`, `<html>`, `<head>`, or `<body>` tags
- [ ] No `fetch()`, `import`, or external URLs
- [ ] No top-level `await` (async code in IIFE)
- [ ] All colors use theme CSS variables
- [ ] Uses `moltamp.poll()` instead of `setInterval`
- [ ] `moltamp.settings.read()` handles null on first run
- [ ] If using Canvas: CSS variables resolved via `getComputedStyle()`
- [ ] API features nested under `"api"` in widget.json
- [ ] Tested with 2+ skins (light and dark)
- [ ] Poll rate >= 1 second
- [ ] Total file size under 20MB

### Screenshots
<!-- Paste a screenshot of your widget in at least one skin -->
