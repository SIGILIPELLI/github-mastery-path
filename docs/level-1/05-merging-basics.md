# 05 · Merging Basics

Once work on a branch is ready, you bring it back into another branch
(usually `main`) with `git merge`. There are two outcomes depending on
whether the branches have diverged: a **fast-forward** merge, or a **true
merge commit**.

## Fast-forward merge

If `main` hasn't moved since you branched off it, Git can simply slide the
`main` pointer forward to match your feature branch — no new commit needed,
because there's nothing to reconcile.

```bash
git switch main
git branch feature/greeting        # branch off main
git switch feature/greeting
echo "feature work" > feature.txt
git add feature.txt
git commit -m "Add feature file"

git switch main
git merge feature/greeting
```
```
Updating 4752e7c..71c4aec
Fast-forward
 feature.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt
```

```bash
git log --oneline --graph --all
```
```
* 71c4aec (HEAD -> main, feature/greeting) Add feature file
* 4752e7c Add second line
* 07246b8 Add notes.txt
```

`main` and `feature/greeting` now point at the exact same commit — history
stays perfectly linear.

## True merge (three-way merge)

If `main` has *also* moved forward since the branch point (someone else
committed, or you did on `main` directly), Git can't just fast-forward. It
creates a new **merge commit** with two parents, combining both histories:

```bash
git switch -c feature/tagline
echo "tagline: ship it" >> notes.txt
git add -A
git commit -m "Add tagline to notes"

git switch main
echo "main update" >> feature.txt
git add -A
git commit -m "Update feature.txt from main"

git merge feature/tagline -m "Merge branch 'feature/tagline'"
```
```
Merge made by the 'ort' strategy.
 notes.txt | 1 +
 1 file changed, 1 insertion(+)
```

```bash
git log --oneline --graph --all
```
```
*   48f9b4e (HEAD -> main) Merge branch 'feature/tagline'
|\
| * 24bd52a (feature/tagline) Add tagline to notes
* | 30e5d21 Update feature.txt from main
|/
* 71c4aec Add feature file
* 4752e7c Add second line
* 07246b8 Add notes.txt
```

Git automatically combined both lines of work because they touched
different files (`notes.txt` vs. `feature.txt`). This automatic combination
is the normal case — most merges need zero human intervention.

## When Git can't merge automatically: conflicts

If both branches changed the *same lines* of the *same file*, Git can't
guess which version is correct and stops, marking the file as conflicted:

```bash
git merge feature/x
```
```
Auto-merging notes.txt
CONFLICT (content): Merge conflict in notes.txt
Automatic merge failed; fix conflicts and then commit the result.
```

At this point Git has written both versions into the file, marked with
conflict markers:

```
<<<<<<< HEAD
main's version of this line
=======
feature/x's version of this line
>>>>>>> feature/x
```

You'd edit the file to keep the correct content, remove the markers, then
`git add` the file and `git commit` to complete the merge. **Full conflict
resolution is covered in depth in Level 2** — for now, just recognize this
output if you see it, and know it means "two branches touched the same
lines; Git needs a human decision."

## `--no-ff`: forcing a merge commit

Sometimes you want a merge commit even when a fast-forward is possible —
to keep an explicit record that a feature branch existed, useful for
tracking what shipped together:

```bash
git merge --no-ff feature/greeting -m "Merge feature/greeting"
```

Many teams standardize on `--no-ff` for all feature-branch merges so the
history always shows branch/merge points clearly in `git log --graph`.

## Aborting a merge

If a merge goes wrong (conflicts you don't want to resolve right now),
back out cleanly:

```bash
git merge --abort
```

This restores the working directory to exactly how it was before you ran
`git merge`.

## Cleaning up after a merge

Once a branch's work has been merged into `main` and you don't need it
anymore:

```bash
git branch -d feature/greeting
```

`-d` (lowercase) refuses to delete a branch with unmerged commits, which is
a good safety net — if it refuses, double-check the branch really is fully
merged before switching to `-D`.

## Exercise

Create two branches, `feature/a` and `feature/b`, both starting from `main`.
On `feature/a`, add a new file `a.txt`. On `feature/b`, add a new file
`b.txt`. Merge both into `main` (in either order) and confirm the second
merge is a true merge commit (not a fast-forward) by checking
`git log --oneline --graph --all`. Delete both feature branches once
merged.
