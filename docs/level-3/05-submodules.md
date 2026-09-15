---
description: "Submodules — A submodule embeds one Git repository inside another — a way to depend on a separate project's source (a shared library, say) while keeping…"
---

# 05 · Submodules

A submodule embeds one Git repository inside another — a way to depend on
a separate project's source (a shared library, say) while keeping its
history completely independent from your own repo's.

## Adding a submodule

```bash
git submodule add /tmp/sub-lib vendor/lib
cat .gitmodules
```
```
[submodule "vendor/lib"]
	path = vendor/lib
	url = /tmp/sub-lib
```

```bash
git status
```
```
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   .gitmodules
	new file:   vendor/lib
```

Two things got staged: `.gitmodules` (a plain text config file listing
every submodule's path and URL) and `vendor/lib` itself — which Git
tracks not as a directory of files, but as a single special entry
recording *which exact commit* of the submodule repo you're pinned to.

```bash
git commit -m "Add lib submodule"
```

## Cloning a repo that has submodules

Submodule contents are **not** fetched by a plain clone:

```bash
git clone /tmp/sub-main sub-main-clone
cd sub-main-clone
ls vendor/lib
```
```
(empty — the directory exists but nothing is in it)
```

You have to explicitly initialize and fetch them:

```bash
git submodule update --init
```
```
Submodule 'vendor/lib' (/tmp/sub-lib) registered for path 'vendor/lib'
Cloning into '.../vendor/lib'...
done.
Submodule path 'vendor/lib': checked out '656f63a7b7717df15ec34518c8e7a929e46768a0'
```

```bash
ls vendor/lib
```
```
lib.txt
```

A shortcut that does both clone steps at once for a fresh checkout:

```bash
git clone --recurse-submodules /tmp/sub-main sub-main-clone
```

## Updating a submodule to a newer commit

The library changes independently:

```bash
cd /tmp/sub-lib
echo "v2" >> lib.txt
git commit -am "lib v2"
```

Pulling that new commit into the submodule checkout, from inside the
parent repo:

```bash
cd vendor/lib
git pull origin main
```
```
Updating 656f63a..aff5ca7
Fast-forward
 lib.txt | 1 +
 1 file changed, 1 insertion(+)
```

```bash
cd ../..
git status
```
```
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   vendor/lib (new commits)
```

The parent repo sees this as **a one-line change**: the pointer commit
for `vendor/lib` moved. Committing it in the parent repo records "from now
on, this project depends on the newer library commit":

```bash
git add vendor/lib
git commit -m "Bump vendor/lib to latest"
```

## Why "modified: vendor/lib (new commits)" and not a file diff

Git deliberately never merges submodule *contents* into the parent
repo's diff — `git diff` on a submodule bump shows only the two commit
SHAs, not the library's internal line changes, because those are a
different repository's history entirely.

## Submodules vs. alternatives

Submodules are notorious for tripping people up (forgetting `--init`,
detached HEAD inside the submodule, confusing "modified" status). For many
use cases, a package manager (npm, Cargo, Go modules) or a monorepo
(Level 4) is a simpler alternative — reach for submodules specifically
when you need an actual separate Git history embedded at an exact commit,
not just a versioned dependency.

## How It Actually Works

- A submodule reference in the parent repo's tree is a special entry
  mode (`160000`, gitlink) whose "content" is not a blob SHA but a
  **commit SHA from a different repository's object database**. This is
  fundamentally different from a normal tree entry — Git deliberately
  does not try to fetch or merge that other repo's objects into the
  parent's own object store automatically.
- `.gitmodules` is the only piece of submodule config actually stored in
  the parent repo's history (it's a regular tracked file); the *checked
  out* clone that ends up on disk at `vendor/lib` is a fully independent
  `.git` repository (or, in newer Git, a `.git` file pointing at
  `.git/modules/vendor/lib` in the parent's own `.git` directory) with its
  own refs, objects, and remotes.
- `git clone` skips submodule content by default specifically because it
  can't assume you want to (or can) reach every referenced submodule
  URL — cloning is not recursive unless told to be, which is the entire
  reason `git submodule update --init` (or `--recurse-submodules`) exists
  as a separate step.
- "modified: vendor/lib (new commits)" appears because the gitlink SHA
  currently checked out inside `vendor/lib` no longer matches the SHA
  recorded in the parent's tree object — from the parent repo's
  perspective this is exactly the same kind of "file changed" detection
  used for any blob, just applied to a commit-SHA pointer instead of file
  bytes.

## Exercise

Create two local repos, add one as a submodule of the other, commit the
addition, then clone the parent fresh into a new directory and confirm
the submodule directory is empty until you run
`git submodule update --init`. Then advance the submodule's own history by
one commit and update the parent's pointer to it in a new commit.
