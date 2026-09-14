# 02 · Interactive Rebase & History Rewriting

Interactive rebase (`git rebase -i`) lets you edit, reorder, squash, or
drop commits before they're shared — the standard tool for turning messy
"wip", "fix typo" commits into a clean, reviewable history.

## Building a messy history

```bash
for i in 1 2 3 4; do
  echo "line $i" >> f.txt
  git add f.txt
  git commit -m "commit $i: WIP fixup typo"
done
echo base0 > base.txt
git add base.txt
git commit -m "base commit 0"

git log --oneline
```
```
27ad367 base commit 0
b48f5a0 commit 4: WIP fixup typo
ead09df commit 3: WIP fixup typo
3f2b264 commit 2: WIP fixup typo
f4026d3 commit 1: WIP fixup typo
```

Five sloppy commits that should really be one or two meaningful ones.

## Opening the interactive rebase

```bash
git rebase -i HEAD~4
```

This opens your editor with a **todo list**, oldest commit first:

```
pick 3f2b264 commit 2: WIP fixup typo
pick ead09df commit 3: WIP fixup typo
pick b48f5a0 commit 4: WIP fixup typo
pick 27ad367 base commit 0
```

Each line is a command plus a commit; editing this list *and saving* is
what performs the rewrite. The common commands:

| Command | Effect |
|---|---|
| `pick` | keep the commit as-is |
| `reword` | keep the commit, but stop to edit its message |
| `edit` | stop right after applying this commit, so you can amend it |
| `squash` | combine into the *previous* commit, keep both messages |
| `fixup` | combine into the previous commit, discard this message |
| `drop` | remove the commit entirely |

Changing lines 2-4 from `pick` to `squash` (scriptable non-interactively
via `GIT_SEQUENCE_EDITOR` for automation/testing) combines those three
commits into the one above them:

```bash
GIT_SEQUENCE_EDITOR="sed -i '' '2,4s/^pick/squash/'" git rebase -i HEAD~4
```
```
Rebasing (2/4)Rebasing (3/4)Rebasing (4/4)
Successfully rebased and updated refs/heads/main.
```

```bash
git log --oneline
```
```
e7a99bd commit 2: WIP fixup typo
f4026d3 commit 1: WIP fixup typo
```

Five commits became two. `git show HEAD` on the squashed commit shows both
original messages concatenated, ready for you to edit down to one clean
summary in your actual editor session (non-scripted use stops here for
exactly that).

## Rewording a commit message

```bash
git rebase -i HEAD~1
# change "pick" to "reword" on the target line, save,
# Git reopens your editor just for the message text
```

Or, for just the very last commit, the shortcut:

```bash
git commit --amend -m "New message"
```

## Reordering commits

Interactive rebase applies commits in the order they appear in the todo
list, top to bottom — simply moving lines around reorders history. This
only works cleanly when the reordered commits don't textually conflict
with each other's changes.

## Splitting a commit

Mark a commit `edit`, let the rebase stop there, then:

```bash
git reset HEAD^          # uncommit, keep changes staged... actually unstaged
git add -p               # stage part of the changes
git commit -m "First half"
git add -A
git commit -m "Second half"
git rebase --continue
```

## The absolute rule

**Never rewrite commits that have already been pushed and might be in
someone else's local history.** Every rewritten commit gets a new SHA
(same mechanism as plain rebase, Level 2), so pushing rewritten history
requires force-pushing, and anyone who already pulled the old commits now
has diverging, duplicated history that's painful to reconcile. Interactive
rebase is for your own **local, unpushed** commits, or a solo branch only
you touch.

## How It Actually Works

- `git rebase -i` works exactly like a plain rebase (Level 2) — it
  replays each commit's diff on a new base and creates new commit
  objects — except it lets you inject extra operations (reorder, combine,
  edit, drop) into that replay sequence instead of always doing a
  straight 1:1 pick.
- The todo list is a real temporary file
  (`.git/rebase-merge/git-rebase-todo`), which is exactly what
  `GIT_SEQUENCE_EDITOR` lets you script against non-interactively — it's
  the same file your normal `$EDITOR` opens, just automated for
  reproducible demonstrations or CI-driven history cleanup.
- `squash`/`fixup` work by *not* creating a separate commit for that line
  at all: Git applies that commit's diff on top of the working tree left
  by the previous pick, then amends the previous commit (new tree, same
  parent, new SHA) to include the combined changes — which is why the
  squashed commit above shows the accumulated diff of three original
  commits but only one new commit object exists at the end.
- `drop`ping a commit or squashing multiple into one doesn't touch the
  *original* commit objects at all — they still exist in the object
  database, simply unreferenced by any branch after the rebase completes.
  This is exactly why `git reflog` (a later module) can recover from an
  interactive rebase gone wrong: the old commits are still sitting in
  `.git/objects` until garbage collection eventually sweeps them.

## Exercise

Create five commits with deliberately bad messages ("wip", "fix", "fix2",
"actually fix", "done"). Use `git rebase -i HEAD~5` to squash them into
one commit with a single clear message describing the net change, and
confirm with `git log --oneline` and `git show --stat` that only one
commit and the correct final diff remain.
