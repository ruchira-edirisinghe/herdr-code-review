# Code review for Herdr

A [Herdr](https://herdr.dev) layout that adds a VS Code-style code view for reviewing changes.

![preview](preview.png)

- **Whole files with inline changes.** Open any changed file from the Changes panel and see
  all of it, not just the hunks. Added lines are green and removed lines red, with old and new
  line numbers, syntax colors and an overview ruler beside the scrollbar. Jump between
  changes with ↑ ↓ (or `n` / `p`, F7 / Shift+F7). "Changes only" folds the unchanged parts.
- **Explorer.** A sidebar with the repo's changed files and its full file tree (from
  `git ls-files`, so ignored files stay out). Folders with changes are marked. Click any file to
  read it, changed or not.
- **Split, code only, terminal only.** The code view opens beside your terminal. One click
  shows only the code or only the terminal; Ctrl+Shift+Z (pane zoom) goes back to the split.
- **Live.** The view follows saves in the repo. VS Code opens at the line you're reading.

## What's in it

| Path | What |
|---|---|
| `layouts/main-review.json` | Herdr's `main` layout, with the Changes block pointing at `review-changes` |
| `mods/review-changes/` | Herdr's Changes panel plus "Open full file" (beside the terminal or in a tab), a ⤢ button per file and double-click |
| `mods/review-file/` | The code view: inline diff, Explorer, Split / Code / Terminal |

Command palette (Ctrl+Shift+P): **Code view beside the terminal** and **Code view in a new tab**
open the Explorer for the repo you're working in.

## Install

Copy `layouts/` and `mods/` into `~/.config/herdr-app` (Windows: `%USERPROFILE%\.config\herdr-app`),
then switch layouts:

```sh
herdr-app check ~/.config/herdr-app/mods/review-file
herdr-app check ~/.config/herdr-app/mods/review-changes
herdr-app do layout.switch '{"name":"main-review"}'
```

Already on your own layout? Keep it, and point its Changes block at the new mod:
`"content": { "mod": "review-changes" }`. Ctrl+Shift+L switches layouts back.

Optional keys for `keys.json`:

```json
{ "mod:review-file": { "ctrl-shift-e": "review-file.explorer", "ctrl-alt-c": "review-file.code-only", "ctrl-alt-s": "review-file.split", "ctrl-alt-t": "review-file.terminal-only" } }
```

## What it runs

Both mods run `git` read-only in the repo you're working in (`status`, `diff`, `ls-files`,
`rev-parse`, with `--no-optional-locks` so nothing is written under `.git`). "Commit" in the
Changes panel runs `git add -A` and `git commit`, as Herdr's own panel does. "VS Code" runs
`code --goto`. Nothing reaches the network.

Built for herdr-app 0.1.18. Tested on Windows 11 ARM64.

## License

Apache-2.0, like herdr-app. `mods/review-changes` is a modified copy of herdr-app's shipped
`changes` mod (see the header of its `main.js` for what changed).
