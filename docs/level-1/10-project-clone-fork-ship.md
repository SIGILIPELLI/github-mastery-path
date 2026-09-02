# 10 · Project — Clone, Fork & Ship a Repo

This capstone pulls together every Level 1 module: local commits, a branch
merged in, remotes, a real GitHub repository, `.gitignore`, and forking/
cloning someone else's project. Work through it top to bottom in order —
each part builds on the last.

## Part 1 — Fork and clone an existing repository

1. On GitHub, find a small, beginner-friendly public repository. GitHub's
   own [`github/gitignore`](https://github.com/github/gitignore) works
   well, or use `octocat/Spoon-Knife` (a repo GitHub maintains specifically
   for practicing forks).
2. Click **Fork** (top-right of the repo page) — this creates a full copy
   under *your* account (`github.com/yourname/Spoon-Knife`).
3. Clone **your fork** (not the original) to your machine:

```bash
git clone https://github.com/yourname/Spoon-Knife.git
cd Spoon-Knife
git remote -v
```
```
origin  https://github.com/yourname/Spoon-Knife.git (fetch)
origin  https://github.com/yourname/Spoon-Knife.git (push)
```

4. Add the original repository as a second remote, conventionally named
   `upstream` — this lets you pull in future updates from the original
   project without affecting your `origin`:

```bash
git remote add upstream https://github.com/octocat/Spoon-Knife.git
git remote -v
```
```
origin    https://github.com/yourname/Spoon-Knife.git (fetch)
origin    https://github.com/yourname/Spoon-Knife.git (push)
upstream  https://github.com/octocat/Spoon-Knife.git (fetch)
upstream  https://github.com/octocat/Spoon-Knife.git (push)
```

This fork → clone → add-upstream pattern is exactly how you'd start
contributing to any real open-source project.

## Part 2 — Build your own project from scratch

Now start an original project that you'll fully own, applying everything
from this level.

```bash
mkdir task-tracker && cd task-tracker
git init -b main
```

Add a `.gitignore` before anything else, so you never accidentally track
files you don't want to:

```bash
cat > .gitignore <<'EOF'
*.log
.DS_Store
.env
EOF
git add .gitignore
git commit -m "Add .gitignore"
```

Create a starting file and commit it:

```bash
cat > README.md <<'EOF'
# Task Tracker

A tiny plain-text task tracker, built while learning Git.
EOF
git add README.md
git commit -m "Add README"

echo "Buy groceries" > tasks.txt
git add tasks.txt
git commit -m "Add first task"
```

## Part 3 — Branch, work, and merge

```bash
git switch -c feature/add-priority-tags
```

Edit `tasks.txt` to add priority markers:

```bash
cat > tasks.txt <<'EOF'
[high] Buy groceries
[low] Reorganize bookshelf
EOF
git add tasks.txt
git commit -m "Add priority tags to tasks"
```

Meanwhile, simulate other work happening on `main` directly (this ensures
your merge is a true three-way merge, not a fast-forward):

```bash
git switch main
echo "See tasks.txt for the current list." >> README.md
git add README.md
git commit -m "Mention tasks.txt in README"

git merge feature/add-priority-tags -m "Merge priority tags feature"
```

Confirm the shape of your history:

```bash
git log --oneline --graph --all
```

You should see a merge commit with two parent lines coming together.

```bash
git branch -d feature/add-priority-tags
```

## Part 4 — Push to a real GitHub repository

1. On GitHub, create a new **empty** repository (no README) named
   `task-tracker`.
2. Connect and push:

```bash
git remote add origin https://github.com/yourname/task-tracker.git
git push -u origin main
```

3. Refresh the repo page on GitHub and confirm: the README, `.gitignore`,
   and `tasks.txt` are all present, and the commit history (visible via the
   "commits" link near the top of the file list) shows every commit
   including the merge.

## Part 5 — One more round-trip

Prove the remote workflow end to end by making one more change and syncing
it:

```bash
echo "[medium] Reply to emails" >> tasks.txt
git add -A
git commit -m "Add another task"
git push
```

Then, simulating "another machine," clone your own repo into a second
folder and confirm everything — including the merge commit — is there:

```bash
cd ..
git clone https://github.com/yourname/task-tracker.git task-tracker-clone
cd task-tracker-clone
git log --oneline --graph
cat tasks.txt
```

## Capstone checklist

By the end of this project you should have, verifiable via `git log
--graph --all` and your GitHub repo page:

- [ ] A local repo with a `.gitignore` committed from the very first commit
- [ ] At least four commits on `main`
- [ ] A feature branch that was merged with a genuine merge commit (two
      parents, not a fast-forward)
- [ ] The branch deleted after merging
- [ ] The repository pushed to a real GitHub repo under your account
- [ ] A second local clone of that same GitHub repo, containing identical
      history
- [ ] A forked copy of someone else's repository, with both `origin` (your
      fork) and `upstream` (the original) configured as remotes

If every box is checked, you're ready for Level 2, where the focus shifts
from "working solo" to "working with other people through GitHub":
resolving real merge conflicts, rebase vs. merge tradeoffs, and running
pull requests end to end.
