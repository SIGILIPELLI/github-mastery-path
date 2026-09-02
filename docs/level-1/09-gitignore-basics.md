# 09 · .gitignore Basics

Not everything in a project folder belongs in version control: compiled
binaries, dependency folders, IDE settings, secrets, and OS-generated
clutter files should stay out of Git entirely. A **`.gitignore`** file
tells Git which paths to never track, so they don't show up in `git
status` and can't be accidentally committed.

## Why this matters

Without a `.gitignore`, a typical project quickly accumulates noise:

```bash
git status --short
```
```
?? node_modules/
?? build/
?? .env
?? .DS_Store
?? *.log
```

Committing `node_modules/` (which can be hundreds of megabytes of files
that `npm install` regenerates anyway) bloats the repository forever —
even deleting it later doesn't remove it from history. Committing `.env`
(which typically holds API keys and database passwords) is a security
incident. `.gitignore` prevents both classes of mistake before they happen.

## Creating a `.gitignore`

It's a plain text file named exactly `.gitignore`, placed at the root of
your repository (you can also have additional ones in subdirectories for
folder-specific rules):

```bash
cat > .gitignore <<'EOF'
# Build output
build/
dist/
*.o

# Dependencies
node_modules/
vendor/

# Environment / secrets
.env
.env.local

# OS files
.DS_Store
Thumbs.db

# Editor settings
.vscode/
.idea/

# Logs
*.log
EOF
```

## Pattern syntax

| Pattern | Matches |
|---|---|
| `build/` | A directory named `build`, anywhere in the repo |
| `*.log` | Any file ending in `.log`, anywhere in the repo |
| `/config.json` | `config.json` only at the repo root (leading `/` anchors it) |
| `docs/*.pdf` | PDFs directly inside `docs/`, not subfolders |
| `**/*.tmp` | `.tmp` files at any depth |
| `!important.log` | Un-ignore this specific file, overriding an earlier broader rule |
| `# comment` | Ignored by Git, useful for documenting sections |

## Verifying it works

```bash
mkdir build
echo "compiled" > build/output.o
git status --short
```
```
(nothing — build/ is ignored, doesn't show up at all)
```

Check whether a specific path would be ignored, and by which rule:

```bash
git check-ignore -v build/output.o
# .gitignore:2:build/    build/output.o
```

## `.gitignore` only affects *untracked* files

A common surprise: if a file was already committed *before* you added it
to `.gitignore`, ignoring it does nothing — Git is already tracking it, and
will keep tracking future changes. You have to explicitly remove it from
tracking:

```bash
git rm --cached .env          # untrack it, but keep the local file on disk
echo ".env" >> .gitignore
git commit -m "Stop tracking .env, add to .gitignore"
```

`--cached` is the key flag — plain `git rm .env` would delete the file from
disk too, which you almost never want here.

## Global gitignore (per-user, all repos)

Some ignores are personal rather than project-specific (your editor's swap
files, your OS's junk files) — set these once, for every repo on your
machine, instead of repeating them in every project's `.gitignore`:

```bash
git config --global core.excludesfile ~/.gitignore_global
cat > ~/.gitignore_global <<'EOF'
.DS_Store
*.swp
.vscode/
EOF
```

## Starting from a template

GitHub maintains a large collection of language/framework-specific
`.gitignore` templates at
[github.com/github/gitignore](https://github.com/github/gitignore) — and
offers to add the right one automatically when you create a new repo
through the GitHub UI (a "Add .gitignore" dropdown, next to the license
picker, on the repo-creation page).

## Worked example

```bash
mkdir myapp && cd myapp
git init -b main

cat > .gitignore <<'EOF'
build/
*.log
.env
EOF

mkdir build && echo "bin" > build/app.bin
echo "SECRET=123" > .env
echo "console.log('hi')" > index.js

git add -A
git status --short
```
```
A  .gitignore
A  index.js
```

Notice `build/` and `.env` never appear — exactly the intended effect.

## Exercise

In a fresh repo, create a `build/` directory with a dummy file in it, an
`.env` file with a fake secret, and a real source file like `main.py`.
Before adding a `.gitignore`, run `git add -A` and `git status` to see all
three tracked. Undo that with `git reset` (unstage everything), then add a
`.gitignore` covering `build/` and `.env`, and confirm with
`git status --short` that only `main.py` and `.gitignore` itself would be
committed.
