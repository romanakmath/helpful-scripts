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

Requires: `pi` (coding agent CLI) and `pi-read-session` on PATH.

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
ln -s "$PWD/pi-read-session" ~/.pi/agent/bin/pi-read-session
```

Note: `~/.pi/agent/bin` must be on your PATH (pi adds it). If a command
is "not found" in an **already-open** terminal after installing, that is
zsh's command cache — run `rehash` once, or just open a new terminal tab.

Note: the shell finds commands via `PATH`, not via your current folder —
being inside `helpful-scripts` makes no difference. To run a script from
its own folder without PATH, use the explicit relative path:
`./pi-session-html`.
