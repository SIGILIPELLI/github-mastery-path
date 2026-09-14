# 08 · Reflog & Recovering Lost Work

`git reflog` is Git's private, local safety net: a log of every place
`HEAD` (and each branch) has pointed, kept independently of your visible
commit history. It's how you recover from an "I just destroyed my work"
moment.

## Losing work on purpose, to prove it's recoverable

```bash
echo a > a.txt; git add .; git commit -m "commit A"
echo b > b.txt; git add .; git commit -m "commit B"
echo c > c.txt; git add .; git commit -m "commit C"
git log --oneline
```
```
9eac5f3 commit C
84d8c5d commit B
18784ec commit A
```

Now the "oops":

```bash
git reset --hard HEAD~2
```
```
HEAD is now at 18784ec commit A
```

```bash
git log --oneline
```
```
18784ec commit A
```

Commits B and C are gone from `git log` — they're unreachable from any
branch. Panic is the normal reaction here, but nothing is actually
deleted yet.

## The reflog still remembers

```bash
git reflog
```
```
18784ec HEAD@{0}: reset: moving to HEAD~2
9eac5f3 HEAD@{1}: commit: commit C
84d8c5d HEAD@{2}: commit: commit B
18784ec HEAD@{3}: commit (initial): commit A
```

Every single place `HEAD` has been — every commit, checkout, reset,
rebase, merge — is listed, newest first, each with a `HEAD@{N}` handle you
can use like any other ref.

## Recovering

```bash
git reset --hard HEAD@{1}
```
```
HEAD is now at 9eac5f3 commit C
```

```bash
git log --oneline
```
```
9eac5f3 commit C
84d8c5d commit B
18784ec commit A
```

Fully recovered — same three commits, same SHAs, because they were never
actually deleted; `reset --hard` had only moved the branch pointer, not
touched the object database.

## When reflog saves you (the common real scenarios)

- An accidental `git reset --hard` that discarded commits.
- A rebase or `commit --amend` you regret — the pre-rebase tip is still in
  the reflog as `HEAD@{n}` for however many operations ago it was.
- A branch you deleted by mistake (`git branch -D feature/x`) — the
  reflog for `HEAD` still has an entry from when you were last on that
  branch; check out that SHA and recreate the branch name with
  `git branch feature/x <sha>`.
- Force-pushed over your own local branch by accident — reflog is
  per-repository and local, so this recovers your local state even though
  the remote branch already changed.

## `git reflog show <branch>` for a specific branch's history

`git reflog` alone shows `HEAD`'s reflog; every branch has its own too:

```bash
git reflog show main
```

## Reflog has a time limit

Entries expire eventually (`git gc` prunes reflog entries older than 90
days by default for reachable commits, 30 days for unreachable ones) —
reflog is a safety net for "recent mistake," not permanent archival. Once
an entry is pruned and the object it referenced becomes truly
unreferenced everywhere, `git gc` can delete the object itself.

## How It Actually Works

- The reflog is stored as plain append-only log files at
  `.git/logs/HEAD` and `.git/logs/refs/heads/<branch>` — one line per ref
  update, recording the old SHA, new SHA, who did it, and a short
  description of the command that caused it. This is a completely
  separate mechanism from the commit graph itself; commits don't know
  they're "in" a reflog, the reflog just records the sequence of pointer
  movements.
- `git reset --hard` (even to an earlier commit) doesn't delete any
  objects — it only rewrites the branch ref (and moves the working
  tree/index to match). The "lost" commits remain fully intact in
  `.git/objects`, simply unreachable from any current branch or tag — the
  exact same state interactive-rebase's discarded originals end up in.
  `HEAD@{1}` is just a friendly alias Git resolves by reading that reflog
  file and picking the SHA from one entry back.
- This is also precisely why the reflog is **local-only and never
  transferred** by push/fetch/clone — it's a record of *your* machine's
  history of ref movements, not part of the shareable object graph, so a
  teammate cloning your repo gets your commits but never your reflog.
- Eventually, `git gc`'s unreachable-object pruning is what actually frees
  the storage — until that runs (and the reflog entries referencing those
  objects have also expired), the data is fully intact and recoverable by
  SHA even without the reflog, via `git fsck --lost-found`.

## Exercise

Make three commits, then run `git reset --hard HEAD~2` to "lose" the last
two. Use `git reflog` to find the SHA of the commit right before the
reset, recover with `git reset --hard <that sha>`, and confirm `git log`
shows all three commits again with their original hashes unchanged.
