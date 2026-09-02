# 04 · Branching Basics

A **branch** in Git is just a movable pointer to a commit. That's it — no
separate copy of the files, no expensive operation. This is why Git
branches are so cheap that the natural workflow is to make a new branch for
every feature, bugfix, or experiment.

## What `main` actually is

When you look at `git log`, you're really following a chain of commits,
each pointing to its parent. A branch name (like `main`) is a label that
always points at the *most recent* commit on that line of work. As you
commit, the branch pointer moves forward automatically.

```
main branch, pointer moves right as you commit:

  A---B---C   <- main
```

## Creating a branch

```bash
git branch feature/greeting
git branch
```
```
  feature/greeting
* main
```

`git branch <name>` creates the branch but does **not** switch to it — the
`*` shows you're still on `main`. The new branch simply points at the same
commit `main` currently points at.

## Switching branches

```bash
git switch feature/greeting
# Switched to branch 'feature/greeting'
```

`git checkout <branch>` does the same thing and is the older, more
overloaded command — `git switch` (added in Git 2.23) was introduced
specifically to make branch switching unambiguous. You'll see both in the
wild; this course uses `git switch` for switching and `git restore` for
discarding file changes, keeping each command single-purpose.

Create and switch in one step:

```bash
git switch -c feature/greeting
# Switched to a new branch 'feature/greeting'
```

(`-c` = "create". The older equivalent is `git checkout -b feature/greeting`.)

## Committing on a branch

```bash
git switch -c feature/greeting
echo "feature work" > feature.txt
git add feature.txt
git commit -m "Add feature file"

git log --oneline --all --graph
```
```
* 71c4aec (HEAD -> feature/greeting) Add feature file
* 4752e7c (main) Add second line
* 07246b8 Add notes.txt
```

Notice `main` is still pointing at `4752e7c` — commits on `feature/greeting`
don't touch `main` at all until you explicitly bring them together (that's
next module, on merging).

## Listing and inspecting branches

```bash
git branch                # local branches, current one marked with *
git branch -a             # local + remote-tracking branches
git branch -v             # each branch + its latest commit
git branch --merged       # branches already merged into the current one
git branch --no-merged    # branches NOT yet merged (careful before deleting these)
```

## Renaming and deleting branches

```bash
git branch -m old-name new-name    # rename a branch
git branch -d feature/greeting     # delete (safe: refuses if unmerged)
git branch -D feature/greeting     # force-delete, even if unmerged
```

## Naming conventions

Git allows almost any branch name, but teams converge on conventions so
intent is obvious at a glance:

| Pattern | Used for |
|---|---|
| `feature/short-description` | New functionality |
| `fix/short-description` or `bugfix/...` | Bug fixes |
| `chore/short-description` | Maintenance, config, tooling |
| `release/1.2.0` | Preparing a release |

Slashes are literal characters to Git (no special meaning), but most tools
render `feature/x` as a nested folder in their branch list, which keeps
long lists organized.

## Worked example

```bash
git switch main
git switch -c fix/typo-in-readme
echo "Fixed typo" >> README.md
git add README.md
git commit -m "Fix typo in README"

git switch main
git log --oneline --all --graph
```
```
* 9f1a2b3 (fix/typo-in-readme) Fix typo in README
* a1b2c3d (HEAD -> main) Add app.py
* 07246b8 Add README
```

`main` (with `HEAD` pointing at it, meaning "the branch you're currently
on") hasn't moved; `fix/typo-in-readme` has one extra commit ahead of it.

## Exercise

Starting from a repo with at least one commit on `main`, create and switch
to a branch called `feature/colors`. On that branch, create a file called
`colors.txt` listing three of your favorite colors, one per line, and
commit it. Switch back to `main` and run `git log --oneline --all --graph`
— confirm you can see both branches and that `main` does not include the
`colors.txt` commit.
