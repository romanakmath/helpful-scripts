# helpful-scripts

Small standalone tools, not related to the trading codebase.

## pi-session-html — find a pi session by phrase, open it as HTML

Pi stores conversations as JSONL under `~/.pi/agent/sessions/`. This script
automates the "I remember a sentence from an old conversation" workflow:

```
pi-session-html "some phrase I remember"
```

1. Greps all session files for the phrase (case-insensitive).
2. Lists matches with date, size, and a context snippet; you pick one.
3. After confirmation, exports the session to HTML (`pi --export`).
4. Opens it in your browser — then Cmd+F to jump to the phrase.

Options:

| Usage | Effect |
|---|---|
| `pi-session-html "phrase" out.html` | write HTML to a specific path instead of `/tmp/pi-sessions/` |
| `pi-session-html "phrase" -y` | no prompts (first match, no confirm) |
| `pi-session-html "phrase" --text` | print readable text in the terminal instead of HTML |

Exports are named `<session-date>_<session-title>.html`, e.g.
`2026-10-03T16-58-02_We-need-a-refactor-of-our-code-most-recnt.html`.
The title comes from the session name (pi-web index), falling back to the
first user message. Re-exporting the same session overwrites its file.

After exporting, the script injects viewer enhancements into the HTML:

- **Thinking blocks** are collapsed by default (`Thinking …`), like Pi Web;
  click a block to expand just that one.
- **Code blocks** are collapsed to a small preview (faded); click the header
  (`▸ python · 42 lines · click to expand`) to open one.
- A floating **`▸ code`** button (bottom right) collapses/expands all code
  blocks at once. The viewer's own toolbar still has *Toggle thinking* and
  *Toggle tools* for global control.

Requires: `pi` (coding agent CLI) and `pi-read-session` on PATH.

## pi-session-import — import a session from a point into Pi Web

Same search flow as `pi-session-html`, but instead of HTML it writes a
trimmed copy of the session (active branch only, starting at the first
entry matching the phrase) into the review folder — one place to edit,
import, and review sessions from different projects:

```
/Users/roman/Dropbox/Code/trading/pi-review-sessions/
```

The folder is symlinked from `~/.pi/agent/sessions/--imports--/` (which
Pi Web scans) and its path is stamped as the session's `cwd`, so Pi Web
groups imports under their own folder **pi-review-sessions** and opens
the agent inside the real folder. Refresh Pi Web and an import appears
within ~30 s, showing your anchor message as its first message.

Note: `pi-review-sessions` sits under the `trading` folder, so sessions
opened there also pick up `trading/AGENTS.md` as context.

Override the label with:
`PI_SESSION_IMPORT_PROJECT=/some/path pi-session-import …`.

```
pi-session-import "Good that improvement was achieved"
pi-session-import "phrase" -y   # no prompts
```

Does not open a browser — it only imports. The picker shows each match's
project directory and message count to help you choose. The script
recreates the imports project directory if it is missing (e.g. after a
reboot, since /private/tmp is wiped).

Pi Web reads `~/.pi/agent/sessions/<folder>/*.jsonl` (one level of
subfolders, symlinks followed). To remove an import from Pi Web again,
delete the file from `pi-review-sessions`.

## pi-sessions — list exported session HTML files, open one

```
pi-sessions            # list newest first, pick one, open in browser
pi-sessions 2          # open the 2nd file in the list directly
pi-sessions -y         # open the newest file, no prompt
pi-sessions -l         # list only, don't open
```

Reads `/tmp/pi-sessions/*.html` (the output of `pi-session-html`). Note the
HTML files live in `/tmp` — macOS clears it on reboot.

## pi-read-session — read a session JSONL as readable text

```
pi-read-session <session.jsonl> [start-phrase] [--all] [--tools]
```

Prints user/assistant messages as clean text. With a start phrase, output
begins exactly at the first message containing it. The active branch is
rendered (same as the Pi Web view); `--all` includes abandoned branches,
`--tools` includes tool calls and results (truncated).

## Install

Both scripts are symlinked into `~/.pi/agent/bin` (already on PATH):

```bash
ln -s "$PWD/pi-session-html" ~/.pi/agent/bin/pi-session-html
ln -s "$PWD/pi-sessions"    ~/.pi/agent/bin/pi-sessions
ln -s "$PWD/pi-read-session" ~/.pi/agent/bin/pi-read-session
```

Note: `~/.pi/agent/bin` must be on your PATH (pi adds it). If a command
is "not found" in an **already-open** terminal after installing, that is
zsh's command cache — run `rehash` once, or just open a new terminal tab.

Note: the shell finds commands via `PATH`, not via your current folder —
being inside `helpful-scripts` makes no difference. To run a script from
its own folder without PATH, use the explicit relative path:
`./pi-session-html`.
