# Recipe: code review for Herdr

Build a VS Code-style review view for uncommitted changes, as one layout and two mods.

## review-file (a mod pane)

- Props, all optional: `{ root, path, on, tab }`. Without `root`, use the repo of the focused
  terminal, else of a terminal in the shown space (`git rev-parse --show-toplevel`).
- Show the whole file as an inline diff: `git diff HEAD --unified=1000000 -- <path>` gives
  every line. Untracked files (or a repo without commits) read from disk as all-added; an
  unchanged file shows plain with a note. Draw rows in a `v_virtual_list`: a colored bar,
  old and new line numbers, a +/− marker and the text, green or red backgrounds.
- Expand tabs and measure columns in one linear pass: whole files are long.
- Color syntax with a small per-line tokenizer (comments, strings, numbers, keywords, calls,
  types) that carries block comments across lines. Tokenize in slices of 500 rows with
  `cx.sleep` between, so long files never hold the window.
- Next/previous change (buttons, n/p, F7/Shift+F7) scroll the change to the center. "Changes
  only" keeps three lines around each change and folds the rest into "⋯ N unchanged lines".
  An overview ruler beside the scrollbar marks each change (green, red, or modified).
- Explorer sidebar (toggle with ☰ or `e`, shared by every copy): the changed files from
  `git status --porcelain -z`, then the tree from `git ls-files -z --cached --others
  --exclude-standard`, folders first, a dot on folders holding changes, the open file
  highlighted and its folders revealed. Plain scrolled lists (rows need `w_full`).
- Split / Code / Terminal: zoom this pane, zoom the tab's terminal, or unzoom
  (`pane.zoom`); split a terminal below when the tab has none. A mod pane isn't told its pane
  id: each copy claims one unclaimed review-file pane (focused first, then the only one left,
  then the only one in the shown tab) and refreshes the claim every tick.
- Watch the repo (`watchFolder`), ignore `.git`, reload 300 ms after a save while the window
  is active, and on refocus. "VS Code" runs `code --reuse-window --goto file:line`.
- Accept `review-file.goto` through `shared()`: `{root, path, on, tab, at}` switches the copy
  in that tab to another file.

## review-changes (clone of the shipped changes mod)

- Add "Open full file" (split right of the focused terminal with `pane.split {mod:
  "review-file", props}`), "In tab" (`tab.new`), a ⤢ button per file and double-click.
  When the tab already has a review-file pane, send `review-file.goto` instead.
- When a mod pane has the focus, keep the space's last repo instead of "No folder".

## main-review layout

`layout.clone` of `main`, with the Changes block's `content.mod` set to `review-changes`.
