# Using these scripts on another server

All scripts work on any machine with the [Pi coding agent](https://github.com/earendil-works)
(`pi` CLI) and, for imports, a running Pi Web. They are plain bash + Python
with no macOS-only dependencies (`open` falls back to `xdg-open` on Linux).

## 1. One-time install

```bash
# get the scripts
git clone https://github.com/romanakmath/helpful-scripts.git
cd helpful-scripts

# put them on PATH
mkdir -p ~/.pi/agent/bin
for f in pi-session-html pi-session-import pi-session-list pi-sessions pi-read-session; do
  ln -sf "$PWD/$f" ~/.pi/agent/bin/$f
done
echo 'export PATH="$HOME/.pi/agent/bin:$PATH"' >> ~/.bashrc   # or ~/.zshrc
# zsh users: run `rehash` once so new commands are found
```

Requirements: `python3` (any 3.8+), `bash`, `pi` CLI.
`pi-session-html` additionally needs `pi-read-session` (same repo, same step).

## 2. Import workflow (pi-session-import) — needs 2 decisions

The script writes imports into a **review folder** and symlinks it into Pi
Web's session scan path so the import shows up in the web UI.

```bash
mkdir -p ~/pi-review-sessions
ln -s ~/pi-review-sessions ~/.pi/agent/sessions/--imports--
```

- **Where the folder lives** — default on this repo author's Mac is a Dropbox
  path; anywhere else the script falls back to `~/pi-review-sessions`
  automatically, or set it explicitly:

  ```bash
  export PI_SESSION_IMPORT_PROJECT=/srv/pi-review-sessions   # optional
  ```

  The folder must **exist** — Pi Web refuses to open a session whose `cwd`
  directory is missing (it runs the agent inside it).

- **Where Pi Web runs** — the script only *writes files*; Pi Web discovers
  them by rescanning `~/.pi/agent/sessions/` every ~30 s. Default URL is
  `http://127.0.0.1:30141` (override with `PI_WEB_URL` if Pi Web listens
  elsewhere). If Pi Web runs on a **different host** than the scripts, point
  its `sessionDir`/scan path at a shared filesystem, or run the scripts on
  the Pi Web host — discovery is file-based, there is no import API.

Auth note: the scripts never talk to Pi Web's API and need no password.
You only need credentials when *checking* the API yourself:

```bash
curl -s -u "pi:<password-from-pi-web.env>" http://127.0.0.1:30141/api/sessions
```

## 3. What works immediately vs what needs setup

| Script | Works immediately? | Needs |
|---|---|---|
| `pi-session-list` | ✅ yes | only `~/.pi/agent/sessions/` to exist (created by pi on first run) |
| `pi-read-session` | ✅ yes | nothing but a session file |
| `pi-sessions` | ✅ yes | `/tmp/pi-sessions/*.html` (empty at first — that's fine) |
| `pi-session-html` | ✅ yes | `pi` CLI for `pi --export`; browser opening needs a GUI (headless server: skip the auto-open, copy the file) |
| `pi-session-import` | ⚠️ 2 steps | the review folder + the `--imports--` symlink (step 2); if Pi Web is on another host, a shared path or run it there |

Sessions themselves are **not** synced between machines — `~/.pi/agent/sessions/`
is local. Copy JSONL files over (rsync/scp) if you want a server-side copy
of conversations that happened on your Mac:

```bash
rsync -av ~/.pi/agent/sessions/ server:~/.pi/agent/sessions/
```

## 4. Smoke test after install

```bash
pi-session-list "anything"          # lists 0 matches in N sessions — proves scan works
pi-session-import "some phrase" -y  # creates an import; appears in Pi Web within ~30 s
pi-session-html "some phrase"       # writes /tmp/pi-sessions/<name>.html
```

## 5. Reminders that apply anywhere

- Import tail: default 50 counted messages (fully visible on open);
  `--N` / `-n N` keeps more, but Pi Web still loads the last 50 first.
- Re-importing the same phrase from the same session **overwrites** the
  previous import (stable id from the output path) — no duplicates.
- Pi Web's folder list = the set of `cwd` values across all indexed
  sessions. Remove a folder by removing the sessions carrying it.
