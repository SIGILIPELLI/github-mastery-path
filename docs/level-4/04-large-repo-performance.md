---
description: "Large Repo Performance (sparse checkout, partial clone) — A repo with years of history and a large working tree is slow in three independent ways: cloning…"
---

# 04 · Large Repo Performance (sparse checkout, partial clone)

A repo with years of history and a large working tree is slow in three
independent ways: cloning (transferring history), checkout (writing files
to disk), and everyday commands like `git log` and `git status` (walking
objects). Each has its own fix.

## Baseline: a repo with real history

```bash
git init -q -b main
for i in $(seq 1 30); do
  echo "line $i" >> file.txt
  git add -A
  git commit -m "commit $i"
done
du -sh .git
```
```
468K	.git
```

Thirty tiny commits already cost nearly half a megabyte of object data —
every commit's tree and the blob's full new snapshot are all separate
objects before Git ever repacks them.

## Shallow clone: truncate history you don't need

```bash
git clone --depth 1 file:///path/to/big shallow-clone
cd shallow-clone
git log --oneline
du -sh .git
```
```
3b850c5 commit 30
124K	.git
```

`--depth 1` fetches only the tip commit and its tree/blobs — not the 29
commits before it. `.git` drops from 468K to 124K. The tradeoff: `git log`
only shows one commit, `git blame` can't walk further back, and pushing
from a shallow clone is restricted. Shallow clones are ideal for CI
checkout jobs that only build the current tip and throw the clone away
after.

## Partial clone: defer blob downloads

```bash
git clone --filter=blob:none file:///path/to/big partial-clone
cd partial-clone
du -sh .git
git log --oneline | wc -l
```
```
warning: filtering not recognized by server, ignoring
128K	.git
      30
```

This is a real, common failure mode worth seeing once: `--filter` requires
the **server** to advertise `uploadpack.allowFilter` over Git's protocol
v2 (GitHub.com supports this; a bare local repo served over the `file://`
transport with default settings does not). When the server doesn't
support it, Git falls back to a normal clone and warns rather than
failing outright. Against a server that does support it (`git config
uploadpack.allowFilter true` on the remote, or GitHub itself),
`blob:none` fetches all commits and trees up front but defers blob
content until a `checkout` or `diff` actually needs it — full history for
`git log`, without downloading every historical blob.

## Committing history in a way that stays fast: `git commit-graph`

```bash
git commit-graph write --reachable
ls .git/objects/info/
```
```
commit-graph
```

The commit-graph file precomputes commit metadata (parents, generation
number, changed-path Bloom filters) in a format `git log`, `git merge-base`,
and `git status` can memory-map and binary-search instead of opening and
zlib-inflating every commit object individually. `git maintenance start`
schedules this (and repacking, and `gc`) to run automatically in the
background instead of as a single slow `git gc` that blocks you.

## Cutting checkout cost with sparse-checkout

Covered in depth in the monorepo module — for large-repo performance
specifically, the relevant number is checkout time: writing 500,000 files
to disk dominates clone time far more than transferring the pack does on
a fast connection. `git sparse-checkout set <dirs>` after a `--filter=tree:0`
clone (which also defers tree objects outside the sparse cone) combines
both fixes: minimal history transferred, minimal files ever written.

## How It Actually Works

- `--depth N` works by having the server perform a graph traversal from
  the requested ref and stop including commits once `N` generations back,
  then send "shallow" boundary markers (stored in `.git/shallow`) instead
  of parent objects for the cut commits. Git treats those parents as
  simply absent — commands that need to walk further (`log`, `blame`,
  `bisect`) hit the boundary and stop, which is why `git fetch
  --unshallow` exists to backfill the missing history later without a
  full re-clone.
- Partial clone (`--filter=blob:none` / `--filter=tree:0` / `--filter=blob:limit=<n>`)
  is a protocol v2 capability: the client tells the server which object
  *types* to omit from the initial packfile, and the server-side
  `upload-pack` process filters them out while still sending every commit
  needed to reconstruct history. Omitted objects aren't gone — they're
  recorded as "promised" (a promisor pack), and any Git command that
  later needs one transparently fetches it over the same remote via a
  lazy `fetch` — this is why a partial clone still "just works" for
  `checkout`, `diff`, and `blame`, just slower the first time a given
  blob is touched.
- The commit-graph's generation numbers give every commit a topological
  "distance from the roots" value, precomputed once. `git merge-base`
  between two commits can then prune huge parts of the graph immediately —
  if commit A's generation number is lower than commit B's, A cannot be
  an ancestor of any commit reachable only through paths higher than B —
  instead of doing a full breadth-first walk of parent pointers every
  time, which is what made `merge-base` and `log --graph` slow on repos
  with tens of thousands of commits before this file existed (Git 2.18+).
- Bloom filters in the commit-graph record, per commit, a probabilistic
  "did this commit touch path X" answer computable without opening the
  commit's tree. `git log -- <path>` uses them to skip inflating trees for
  commits the filter says definitely didn't touch that path, falling back
  to an exact tree-diff only for commits the filter can't rule out (a
  Bloom filter can false-positive but never false-negative).

## Exercise

In a fresh repo with 20+ commits, run `git clone --depth 5` from it and
confirm with `git log --oneline` that exactly 5 commits are present. Then
run `git fetch --unshallow` in the clone and confirm `git log --oneline`
now shows the full history, checking `.git/shallow` disappears afterward.
