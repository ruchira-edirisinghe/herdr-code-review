# Code review for Herdr

A [Herdr](https://herdr.dev) layout that adds a VS Code-style code view for reviewing changes,
plus agent tracking inspired by [cmux](https://cmux.com). It is also part of
[Zendr](https://github.com/ruchira-edirisinghe/zendr), a Windows ARM64 build of Herdr's desktop app.

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
- **A space is explorer, code reviewer and terminal.** Right-click a tab to show or hide each
  of the three. Each space remembers what it shows.

Inspired by cmux:

- **Notifications inbox.** A bell with an unread count in the tab bar. Each time an agent needs
  you or finishes, each session-bridge message and every other toast is listed, grouped by day,
  each kind in words with an icon. Filter them, remove one, or pause them all for an hour.
  Ctrl+Shift+I opens it; Ctrl+Shift+U jumps to the latest unread.
- **Notification settings.** The mods follow `notifications` in settings.json: for each kind
  (`blocked`, `done`, `bridge`, `other`) `popup` and `inbox` (and `sound`), plus `quiet_until`,
  `badge` (`count`, `dot`, `off`), `keep`, `read_on_visit`, `rings` and `sidebar_note`. In
  [Zendr](https://github.com/ruchira-edirisinghe/zendr), Settings → Notifications sets them, and
  the app follows pop-up length, do not disturb and sounds by kind.
- **Attention rings.** A pane whose agent needs you gets a tinted header. Its tab lights up.
- **Sidebar line.** Under each space: its listening ports, its branch's pull request (`gh`), its
  [session-bridge](https://github.com/iddhi-sulakshana/claude-mods/tree/main/claude-session-bridge)
  status and its latest unread notification.
- **Agent summaries.** The Agents list shows each agent's current task and a one-line summary
  (the question it asks, the step it is on, or its last message).

New here:

- **The hub.** A panel on the right with every agent and what it's doing. The coordinator (a
  Claude Code session that plans the work and messages the others over the session bridge) is
  shown apart at the top. It lists the messages between sessions and has a box to message the
  coordinator, one agent or everyone as you. **Start coordinator** opens one with its brief.
- **Session colours.** Give a session one of twelve named colours, or let it use its space's.
  The colour tints its row in Agents, its tab and its hub card (stronger when selected). Right-click the row,
  the card or the tab (Colour…), or press Ctrl+Shift+Y. A space's menu sets the space's colour.

Inspired by Warp:

- **Command search** (Ctrl+Shift+R, or the `</>` button in the tab bar). Shell history and saved
  workflows in one list. Enter pastes the command into the terminal to check; Ctrl+Enter runs it.
- **Workflows.** Saved commands with `{{placeholders}}`, in `workflows.json` (save one from history).
- **Ask AI** (Ctrl+Shift+Space). Describe it; Claude Code (`claude -p`) writes the command.
- **Launch configurations** (Ctrl+Shift+J). Save the open spaces and tabs as a setup and reopen it.

## What's in it

| Path | What |
|---|---|
| `layouts/main-review.json` | Herdr's `main` layout, with the Changes block pointing at `review-changes` |
| `mods/review-changes/` | Herdr's Changes panel plus "Open full file" (beside the terminal or in a tab), a ⤢ button per file and double-click |
| `mods/review-file/` | The code view: inline diff, Explorer, Split / Code / Terminal |
| `mods/review-tabs/`, `mods/review-menu/` | Herdr's tab bar and menu, plus the bell, the per-space Show/Hide items and session-bridge requests |
| `mods/inbox/` | The notifications inbox |
| `mods/review-toast/`, `mods/review-pane/` | Herdr's toast and pane header: toasts go into the inbox; panes ring |
| `mods/review-spaces/`, `mods/review-agents/` | Herdr's sidebar and agents list, with the extra lines |
| `mods/hub/` | The hub: the coordinator, every agent, the session-bridge messages and a message box |
| `mods/session-color/` | The colour picker for sessions and spaces |
| `mods/commands/`, `mods/launch/` | Command search (history, workflows, Ask AI) and launch configurations |

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

Keys for `keys.json` (the inbox's are under `global`):

```json
{ "global": { "ctrl-shift-i": "review-tabs.inbox", "ctrl-shift-u": "review-tabs.jump-unread", "ctrl-shift-y": "review-tabs.color",
    "ctrl-shift-r": "review-tabs.command-search", "ctrl-shift-space": "review-tabs.ask-ai", "ctrl-shift-j": "review-tabs.launch" },
  "mod:review-file": { "ctrl-shift-e": "review-file.explorer", "ctrl-alt-c": "review-file.code-only", "ctrl-alt-s": "review-file.split", "ctrl-alt-t": "review-file.terminal-only" } }
```

## What it runs

The code view and Changes panel run `git` read-only in the repo you're working in (`status`, `diff`, `ls-files`,
`rev-parse`, with `--no-optional-locks` so nothing is written under `.git`). "Commit" in the
Changes panel runs `git add -A` and `git commit`, as Herdr's own panel does. "VS Code" runs
`code --goto`. The sidebar runs `gh pr view` for spaces on a branch, and, on Windows, one
PowerShell look at listening ports every 20 seconds while the window is active.

Built for herdr-app 0.1.18. Tested on Windows 11 ARM64.

## License

Apache-2.0, like herdr-app. The `review-*` mods other than `review-file` are modified copies of
herdr-app's shipped mods (their `mod.json` names which one, under `cloned_from`).
