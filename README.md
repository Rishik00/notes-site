# Reading Notes — static site

The public site that ReadXiv publishes notes to. It is a plain static site
(no build step) so it can be hosted free on GitHub Pages.

## Files

| File            | Role                                                              |
| --------------- | ----------------------------------------------------------------- |
| `index.html`    | Lists every published note, read from `manifest.json`.            |
| `note.html`     | The one template. `note.html?id=<arxivId>` fetches `notes/<id>.md`, renders the markdown, and typesets equations. |
| `style.css`     | All styling (light + dark).                                       |
| `manifest.json` | Index of published notes (id, title, authors, tags, date).        |
| `notes/*.md`    | The published note files, copied here by ReadXiv on publish.      |

Markdown is rendered in the browser with **markdown-it**; equations with
**MathJax** (`$...$` inline, `$$...$$` display, plus `\(...\)` / `\[...\]`).
Both load from a CDN — no install needed.

## How publishing works

ReadXiv's publish button (in the reader) does this automatically:

1. Copies the note's `.md` into `notes/`.
2. Adds/updates its entry in `manifest.json`.
3. `git add` + commit, then pushes via the `gh` CLI.
4. GitHub Pages redeploys the site.

## One-time setup

```bash
# from this folder
gh auth login                 # authenticate once
gh repo create notes-site --public --source=. --push
# then enable Pages:
gh api -X POST repos/:owner/notes-site/pages -f source.branch=main
```

After that you never touch git by hand — ReadXiv pushes for you.

## Preview locally

```bash
python -m http.server 8080
# open http://localhost:8080
```
