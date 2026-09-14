# 06 · Tags & Releases

Tags mark a specific commit permanently — most often "this is version
1.0.0." GitHub Releases build on top of tags to add release notes,
binaries, and a page users can find.

## Lightweight vs annotated tags

```bash
git tag lightweight-v1
git tag -a v1.0.0 -m "First release"
git tag -l
```
```
lightweight-v1
v1.0.0
```

They look the same in `git tag -l`, but they're different object types:

```bash
git cat-file -t v1.0.0
git cat-file -t lightweight-v1
```
```
tag
commit
```

`v1.0.0` is its own object (type `tag`) with a message and tagger
identity; `lightweight-v1` is nothing but a ref pointing directly at a
commit — no separate object at all.

```bash
git show v1.0.0
```
```
tag v1.0.0
Tagger: You <you@example.com>
Date:   Fri Sep 11 10:26:03 2026 +0530

First release

commit 4284e2119776d33fa9fd59891fab801f0d72991c
Author: You <you@example.com>
...
```

`git show` on an annotated tag prints the tag's own metadata *and then*
the commit it points to — proof there are two objects involved, not one.

**Use annotated tags for releases.** They carry a real identity (tagger,
date, message) and are what `git describe` and GitHub Releases expect;
reserve lightweight tags for quick personal bookmarks.

## Pushing tags

Tags are not pushed automatically with `git push` — they need to be
pushed explicitly:

```bash
git push origin v1.0.0          # one tag
git push origin --tags          # all tags
```

## Semantic versioning

The near-universal convention is `MAJOR.MINOR.PATCH`:

- **MAJOR** — breaking/incompatible changes
- **MINOR** — new backward-compatible functionality
- **PATCH** — backward-compatible bug fixes

`v1.0.0 → v1.1.0` adds a feature with no breakage; `v1.1.0 → v2.0.0` may
break existing users' code; `v1.1.0 → v1.1.1` is a bugfix only.

## Creating a GitHub Release

```bash
gh release create v1.0.0 --title "v1.0.0" \
  --notes "Initial public release." \
  dist/app.tar.gz
```

From `gh release create --help`:

```
If a matching git tag does not yet exist, one will automatically get created
from the latest state of the default branch.
Use `--target` to point to a different branch or commit for the automatic tag creation.
Use `--verify-tag` to abort the release if the tag doesn't already exist.
To fetch the new tag locally after the release, do `git fetch --tags origin`.

To create a release from an annotated git tag, first create one locally with
git, push the tag to GitHub, then run this command.
Use `--notes-from-tag` to get the release notes from the annotated git tag.
```

Two workflows exist: let `gh release create` auto-create the tag on
whatever branch/commit you point it at, or (recommended for real
projects) create and push the annotated tag yourself first, then run
`gh release create v1.0.0 --notes-from-tag` to pull the tag message in as
the release notes automatically.

## Auto-generated release notes

GitHub can build notes from merged PRs since the last tag:

```bash
gh release create v1.1.0 --generate-notes
```

This groups merged PRs by label (bugs, features, etc.) into a changelog —
configurable via a `.github/release.yml` categorization file.

## How It Actually Works

- A lightweight tag is literally a file (or a packed-refs line) at
  `refs/tags/<name>` containing nothing but a commit SHA — functionally
  identical to a branch ref, except Git/GitHub treat `refs/tags/*` as
  immutable by convention (nothing moves a tag forward the way commits
  move a branch forward).
- An annotated tag is a real object in the object database, of type
  `tag`, containing: the SHA it points to, the "tagged object" type, the
  tagger's name/email/date, and a message — with its **own SHA**, computed
  over that content the same way commit/blob/tree SHAs are computed. The
  ref at `refs/tags/v1.0.0` points to the *tag object's* SHA, not directly
  to the commit; the tag object then points to the commit. That extra
  layer of indirection is exactly what `git cat-file -t` and the two-part
  `git show` output revealed above.
- `git push` excludes tags by default specifically because tags are often
  local/experimental bookmarks; GitHub Releases build on the *pushed* tag,
  so a release always corresponds to a specific, permanent, shareable
  commit — which is the whole point of tagging a release in the first
  place rather than just pointing people at a branch that keeps moving.
- A GitHub Release is a database record (title, notes, assets, "latest"
  flag) associated with a tag name; deleting a release does not delete
  the underlying git tag, and deleting the tag does not delete the
  release record — they're linked but independently managed, which is
  why GitHub's UI has separate delete actions for each.

## Exercise

Create an annotated tag `v0.1.0` on your scratch repo's latest commit,
push it, and use `gh release create v0.1.0 --notes-from-tag` to publish a
release whose notes come from the tag's own annotation message. Confirm
with `gh release view v0.1.0` that the notes match what you wrote in
`git tag -a`.
