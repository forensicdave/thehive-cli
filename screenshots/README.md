# Screenshots

These images are referenced by the **Interactive editing** section of the main
[`../README.md`](../README.md). Drop PNGs here with **these exact filenames** so the
links in the README resolve:

| File | Shows | Command to capture |
|------|-------|--------------------|
| `interactive-case.png` | Editing a case's title/description in `$EDITOR` | `thehive-cli -t 666 --interactive` |
| `interactive-page.png` | Creating/editing a case page (Title/Category/Order + markdown body) | `thehive-cli -t 666 --add-page --interactive` |
| `interactive-observable.png` | The line-based IOC table | `thehive-cli -t 666 --add-observable --interactive` |
| `interactive-comment.png` | Composing a markdown comment | `thehive-cli -t 666 --add-comment --interactive` |
| `interactive-softLocking.png` | Shows how editor handles concurrent modification of case | `thehive-cli -t 666 --interactive` |

## Capture tips

- Show the terminal with your `$EDITOR` **open on the front-matter buffer** — that's the highlight of the feature.
- PNG, roughly 900–1200 px wide; crop to the terminal window. On macOS use **⌘⇧4** for a region grab (or **⌘⇧5**).
- If a shot comes out very wide and renders too large on GitHub, swap the markdown in the main README for an HTML tag to cap it, e.g. `<img src="screenshots/interactive-observable.png" alt="IOC table" width="900">`.

## Uploading (web-only)

On the repo page: **Add file → Upload files**, then drag this whole `screenshots/`
folder (with your PNGs inside) into the drop area — GitHub preserves the folder structure.

Once the four images are in place you can delete this placeholder file, or keep it as a guide.
