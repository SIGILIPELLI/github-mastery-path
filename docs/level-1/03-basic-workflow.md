# 03 · The Basic Workflow (init/add/commit/status/log)

Every Git repository, no matter how complex it eventually becomes, is built
from the same five commands. Get comfortable with these and you already
know most of what you'll type day to day.

## `git init` — start tracking a project

```bash
mkdir demo && cd demo
git init -b main
# Initialized empty Git repository in /path/to/demo/.git/
```

`git init` creates a hidden `.git/` folder inside the current directory —
that folder *is* the repository (all history, branches, and configuration
live there). `-b main` names the initial branch `main` (if you set
`init.defaultBranch` globally in the previous module, you can drop this
flag).

## The three areas

Understanding these three areas is the single most useful mental model in
Git:

```
Working Directory  --git add-->  Staging Area  --git commit-->  Repository (history)
   (your files)                   (the "index")                  (.git/ objects)
```

- **Working directory** — the actual files you see and edit on disk.
- **Staging area** (a.k.a. "the index") — a holding area where you build up
  exactly what the *next* commit will contain. This is what makes Git
  different from "just saving": you choose precisely which changes go into
  each commit, even if you've changed ten files.
- **Repository** — the permanent, committed history, stored as objects
  inside `.git/`.

## `git status` — what's changed, right now

```bash
echo "hello" > notes.txt
git status
```
```
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        notes.txt

nothing added to commit but untracked files present (use "git add" to track)
```

`git status` is safe to run constantly — it changes nothing, it only
reports. Get in the habit of running it before and after every step while
you're learning.

## `git add` — stage changes

```bash
git add notes.txt
git status --short
```
```
A  notes.txt
```

`--short` gives a compact view: `A` means "added to the staging area" (new
file staged). You can stage multiple files, a whole directory, or
everything at once:

```bash
git add file1.txt file2.txt   # specific files
git add src/                  # a whole directory
git add -A                    # everything changed, anywhere in the repo
git add .                     # everything changed, from the current dir down
```

## `git commit` — save a snapshot

```bash
git commit -m "Add notes.txt"
```
```
[main (root-commit) 07246b8] Add notes.txt
 1 file changed, 1 insertion(+)
 create mode 100644 notes.txt
```

`-m "..."` supplies the commit message inline. Without `-m`, Git opens your
configured editor so you can write a longer message. `07246b8` is the first
seven characters of this commit's SHA-1 hash — a unique fingerprint of the
commit's content, its metadata, and its parent commit.

**Writing good commit messages:** first line under ~50 characters,
imperative mood ("Add", "Fix", "Refactor" — not "Added" or "Adding"), and a
blank line before any additional detail:

```bash
git commit -m "Fix off-by-one error in pagination" -m "The last page was
being dropped when total items was an exact multiple of page size."
```

## `git log` — see the history

```bash
echo "second line" >> notes.txt
git add -A
git commit -m "Add second line"

git log
```
```
commit 4752e7c1a2b3... (HEAD -> main)
Author: Ada Lovelace <ada@example.com>
Date:   Tue Sep 1 10:00:00 2026 -0700

    Add second line

commit 07246b8f9e8d... 
Author: Ada Lovelace <ada@example.com>
Date:   Tue Sep 1 09:55:00 2026 -0700

    Add notes.txt
```

Useful variants:

```bash
git log --oneline              # one line per commit, compact
git log --oneline --graph      # add an ASCII graph of branches/merges
git log -p                     # show the full diff for each commit
git log -3                     # only the last 3 commits
git log --author="Ada"         # filter by author
git log notes.txt              # only commits that touched this file
```

## `git diff` — see exactly what changed

```bash
echo "third line" >> notes.txt
git diff              # unstaged changes vs. the last commit
```
```
diff --git a/notes.txt b/notes.txt
index 27019c3..a1b2c3d 100644
--- a/notes.txt
+++ b/notes.txt
@@ -1,2 +1,3 @@
 hello
 second line
+third line
```

```bash
git add notes.txt
git diff              # now shows nothing -- change is staged
git diff --staged     # shows staged changes vs. the last commit
```

## Putting it all together

```bash
git init -b main
git config user.name "Ada Lovelace"        # if not already set globally
git config user.email "ada@example.com"

echo "# My Project" > README.md
git add README.md
git commit -m "Add README"

echo "print('hello')" > app.py
git status --short
# ?? app.py
git add app.py
git commit -m "Add app.py"

git log --oneline
# a1b2c3d Add app.py
# 07246b8 Add README
```

## Exercise

Create a new folder, run `git init -b main` inside it, and set your
name/email locally if you haven't globally. Create a file called
`todo.txt` with three lines, each on its own line, added one at a time —
so you make three separate commits, each adding one line (use `git status`
and `git diff` before each commit to see what's about to be committed).
Finish with `git log --oneline` and confirm you see exactly three commits.
