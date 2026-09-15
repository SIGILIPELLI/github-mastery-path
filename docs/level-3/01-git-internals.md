---
description: "Git Internals (objects, refs, .git directory) — Everything Git does — commits, branches, merges, diffs — is built from four object types stored as…"
---

# 01 · Git Internals (objects, refs, .git directory)

Everything Git does — commits, branches, merges, diffs — is built from
four object types stored as content-addressed files. This module opens up
`.git/` directly and shows the real objects behind one commit.

## Making one commit and looking inside `.git/objects`

```bash
git init -q -b main
echo "hello" > a.txt
git add a.txt
git commit -m "Add a.txt"

find .git/objects -type f
```
```
.git/objects/ce/013625030ba8dba906f756967f9e9ca394464a
.git/objects/1e/de661402a0456758d54fcea650d9abdd82ee81
.git/objects/2e/81171448eb9f2ee3821e3d447aa6b2fe3ddba1
```

One commit produced **three objects**, not one. Every object lives at
`.git/objects/<first 2 hex chars>/<remaining 38 chars>` — the split into a
subdirectory is purely a filesystem optimization so no single directory
ends up with millions of entries.

## The blob: your file's content

```bash
git hash-object a.txt
```
```
ce013625030ba8dba906f756967f9e9ca394464a
```

```bash
git cat-file -p ce013625030ba8dba906f756967f9e9ca394464a
```
```
hello
```

A **blob** stores file content only — no filename, no permissions,
nothing but bytes. `hash-object` computed this SHA before the file was
even added; the object at that path in `.git/objects` is the exact same
content, compressed with zlib.

## The tree: a directory snapshot

```bash
git cat-file -p 2e81171448eb9f2ee3821e3d447aa6b2fe3ddba1
```
```
100644 blob ce013625030ba8dba906f756967f9e9ca394464a	a.txt
```

A **tree** maps names to blobs (files) or other trees (subdirectories),
each with a file mode (`100644` = regular file, `100755` = executable,
`040000` = subdirectory). A repo with nested folders has one tree object
per directory level, each nested tree referencing its parent's entry by
SHA.

## The commit: a pointer plus metadata

```bash
git cat-file -p $(git rev-parse HEAD)
```
```
tree 2e81171448eb9f2ee3821e3d447aa6b2fe3ddba1
author You <you@example.com> 1789102801 +0530
committer You <you@example.com> 1789102801 +0530

Add a.txt
```

A **commit** object points at exactly one tree (the whole project's
snapshot at that moment), zero or more parent commits (none here — it's
the first commit), and carries author/committer identity, timestamps, and
the message. Notice: no diff is stored anywhere. Git computes diffs on
demand by comparing two trees; it never stores a delta as the primary
representation.

## The fourth type: refs, and why they're just files

```bash
cat .git/refs/heads/main
cat .git/HEAD
```
```
<the commit's full SHA>
ref: refs/heads/main
```

A branch is nothing more than a 41-byte text file containing a SHA.
`HEAD` is one level of indirection further: it usually holds
`ref: refs/heads/main` (you're "on" a branch), so moving to a new commit
via `git commit` just means Git updates `refs/heads/main` to the new
commit's SHA and leaves `HEAD` pointing at the same ref name.

## Detached HEAD: when the indirection breaks

```bash
git checkout $(git rev-parse HEAD)
cat .git/HEAD
```
```
<the commit's full SHA>
```

Now `HEAD` holds a raw SHA instead of `ref: refs/heads/main` — you're in
"detached HEAD" state. Any new commit here still gets created normally,
but no branch ref moves to point at it, so it becomes unreachable (and
eventually garbage-collected) the moment you switch away, unless you
create a branch to keep pointing at it first (`git switch -c rescue`).

## Walking the graph with `git log --format=raw`

```bash
git switch -c main
git log --format=raw
```
```
commit 4b1c...
tree 2e81171448eb9f2ee3821e3d447aa6b2fe3ddba1
author You <you@example.com> 1789102801 +0530
committer You <you@example.com> 1789102801 +0530

    Add a.txt
```

This is the same content `cat-file -p` showed, confirming `git log` is
just walking the parent chain of commit objects and printing each one —
there's no separate "history" data structure beyond the commits'
`parent` pointers themselves.

## How It Actually Works

- Every object's identity **is** the SHA-1 (or SHA-256, on repos
  initialized with the newer hash algorithm) hash of its own
  zlib-compressed content, prefixed with a small header
  (`"blob <size>\0"`, `"tree <size>\0"`, `"commit <size>\0"`). This is why
  Git is called "content-addressed storage": you cannot change an
  object's content without changing its address (SHA), which is the
  entire basis of Git's integrity guarantees — a corrupted or tampered
  object simply won't hash to the name it's stored under, and `git
  fsck` catches this immediately.
- Because objects are addressed by content, **identical content is
  automatically deduplicated**: two files with byte-identical content in
  the same or different commits share exactly one blob object; two
  identical directory listings share exactly one tree object. This is why
  committing a file that reverts to a previous exact state costs no new
  storage.
- A commit's tree is a *complete* snapshot, not a diff, but Git makes this
  cheap via trees sharing unchanged subtrees between commits — if you
  change one file in a deep directory, only that file's blob, that
  directory's tree, and every parent directory's tree up to the root need
  new objects; every sibling directory's tree object is reused unchanged.
- Refs are deliberately "dumb": a loose ref is just a file whose entire
  content is a 40/64-character hex SHA (or another ref name, for symbolic
  refs like `HEAD`). This simplicity is why so much of Git's higher-level
  behavior (branches, tags, remote-tracking branches) is really just
  "which file in `.git/refs/...` points where," and why tools can
  manipulate Git repos correctly without needing a full Git implementation
  — reading/writing these files by hand (carefully) works.

## Exercise

In a fresh scratch repo, make two commits touching the same file. Use
`git cat-file -p HEAD` and `git cat-file -p HEAD^` to inspect both commit
objects, note that they reference two *different* tree SHAs, and use
`git cat-file -p` on each tree to see exactly which blob SHA changed
between the two snapshots.
