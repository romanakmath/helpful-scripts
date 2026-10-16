# GitHub — publish this repository (cheat sheet)

One-time setup (already done once your git identity is configured):

```bash
gh auth login
# choose: GitHub.com → HTTPS → Login with a web browser, follow the prompt
```

## Publish a new folder as a public repo (the short way)

```bash
cd /path/to/my-new-repo
git init
git add .
git commit -m "Initial commit"
gh repo create my-new-repo --public --source=. --push
```

`gh repo create --source=. --push` creates the empty repo on GitHub,
adds it as `origin`, and pushes the current branch in one go.

## The manual way (without gh)

1. On github.com: **New repository** → name it → **Public** → do NOT
   initialize with README/.gitignore/license (keep it empty) → Create.
2. Locally:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<username>/my-new-repo.git
git push -u origin main
```

## Everyday cycle afterwards

```bash
git add .
git commit -m "what changed"
git push
```

## Useful extras

```bash
gh auth status                      # am I logged in? as whom?
gh repo view --web                  # open the repo page in browser
git remote -v                       # show configured remote
git status                          # what is staged/modified
```

Notes:
- This folder is its own git repo, independent of the trading worktrees.
- Public repo = visible to everyone: never commit keys, tokens, DBs, or
  session exports.
