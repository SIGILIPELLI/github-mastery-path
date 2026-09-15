---
description: "Cherry-Picking — git cherry-pick applies one specific commit from anywhere in the repo onto your current branch — useful for grabbing a single fix without…"
---

# 03 · Cherry-Picking

`git cherry-pick` applies **one specific commit** from anywhere in the
repo onto your current branch — useful for grabbing a single fix without
merging or rebasing an entire branch.

## Setting up a fix that needs to land in two places

```bash
git switch -c hotfix
echo "urgent fix" >> f.txt
git commit -am "Fix critical bug in parser"

git switch main
echo "unrelated" >> g.txt
git add g.txt
git commit -m "Unrelated main work"

git log --oneline --all
```
```
a2f432c Fix critical bug in parser
92ed2ce Unrelated main work
741e584 base
```

The hotfix commit exists only on `hotfix`. You want just that one fix on
`main` right now, without merging all of `hotfix` (which might contain
other unfinished work).

## Cherry-picking it

```bash
git cherry-pick a2f432c
```
```
[main 8583c4d] Fix critical bug in parser
 Date: Fri Sep 11 10:31:14 2026 +0530
 1 file changed, 1 insertion(+)
```

```bash
git log --oneline --graph --all
```
```
* a2f432c Fix critical bug in parser
| * 8583c4d Fix critical bug in parser
| * 92ed2ce Unrelated main work
|/
* 741e584 base
```

Read this graph carefully: `main` now has **its own new commit**
(`8583c4d`) with the same message and same diff as `hotfix`'s commit
(`a2f432c`), but a **different SHA**, a different parent, and a different
author date-of-application. It's a copy of the change, not a reference to
the original commit.

## Common real-world use: backporting to a release branch

```bash
git switch release/2.x
git cherry-pick <sha-of-fix-on-main>
```

This is the standard way security/bug fixes land on older maintained
release branches without dragging in every unrelated commit that's
happened on `main` since the branch diverged.

## Cherry-picking a range

```bash
git cherry-pick abc123..def456   # every commit after abc123 up to def456, in order
git cherry-pick abc123^..def456  # same range, but including abc123 itself
```

## Conflicts during cherry-pick

Same markers, same recovery pattern as merge/rebase:

```bash
git cherry-pick a2f432c
# CONFLICT (content): Merge conflict in f.txt
# ... resolve, then:
git add f.txt
git cherry-pick --continue
# or back out entirely:
git cherry-pick --abort
```

## `-x`: recording where it came from

```bash
git cherry-pick -x a2f432c
```

Appends `(cherry picked from commit a2f432c...)` to the new commit's
message — valuable on release branches so anyone reading history later
can trace a backported fix back to its origin commit.

## How It Actually Works

- A cherry-pick computes the diff between the target commit and *its own
  parent* (exactly one commit's worth of change, isolated from everything
  else on that branch), then applies that diff to your current working
  tree/index — the same "compute diff, apply diff" machinery rebase uses
  per commit, just invoked for a single arbitrary commit instead of a
  whole sequence.
- The new commit gets a brand-new tree (reflecting your branch's state
  plus that one diff) and **one parent**: your branch's previous tip —
  nothing about the original commit's parent or branch history carries
  over. This is why cherry-picking never creates a merge commit and why
  the new commit's SHA is unrelated to the source commit's SHA even
  though the message and diff match.
- Conflicts happen for the identical reason they do in a merge: the diff
  being applied assumes context (surrounding lines) that may not match
  your branch's current file content, and Git can't reconcile that
  automatically.
- `-x`'s appended trailer is pure metadata in the commit message string —
  Git attaches no special object-level link between the two commits.
  Tools that need to find "was this fix backported yet" typically grep
  commit messages for that trailer text (or the original SHA), since
  there's no structural graph connection between a commit and its
  cherry-picked copies.

## Exercise

Create a `main` branch and a `release/1.x` branch from the same base.
Commit a bug fix only on `main`. Cherry-pick that exact commit onto
`release/1.x` using `-x`, and confirm via `git log` that the backported
commit's message includes the original SHA it was picked from.
