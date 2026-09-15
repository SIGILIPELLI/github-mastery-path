---
description: "Release Engineering & Versioning Strategy — The type prefix (feat, fix, chore, docs, ...), optional ! or a BREAKING CHANGE: footer, and optional (scope)…"
---

# 07 · Release Engineering & Versioning Strategy

A version number is a promise to consumers about what kind of change to
expect. This module builds that promise mechanically: Conventional
Commits as the input, SemVer as the output, and Git tags plus GitHub
Releases as the artifact — all derivable from `git log`, not from a
human deciding "this feels like a minor bump."

## Conventional Commits: making history machine-readable

```bash
git log --oneline
```
```
615405e Add CODEOWNERS
5bd6fe0 Small fix behind flag (squashed)
b6dfdf4 Merge release 1.1
8739f20 Prepare 1.1
79a9272 Initial release
0ff1e35 Add feature 1
```

None of these messages are machine-parseable for version bumping. A
Conventional Commits-disciplined history looks like:

```
feat(api): add pagination to /users endpoint
fix(web): correct off-by-one in date picker
feat!: drop support for Node 16

BREAKING CHANGE: minimum Node version is now 18
```

The `type` prefix (`feat`, `fix`, `chore`, `docs`, ...), optional `!` or a
`BREAKING CHANGE:` footer, and optional `(scope)` are the entire input a
release tool needs to compute the next version without a human deciding.

## SemVer: what each commit type maps to

| Commit type | Version bump | Example |
|---|---|---|
| `fix:` | patch (`1.2.0` → `1.2.1`) | bug fix, no API change |
| `feat:` | minor (`1.2.0` → `1.3.0`) | new backward-compatible capability |
| `feat!:` / `BREAKING CHANGE:` footer | major (`1.2.0` → `2.0.0`) | breaks existing consumers |
| `chore:`, `docs:`, `style:`, `refactor:`, `test:` | no bump | no user-facing change |

## Tagging a release for real

```bash
git tag -a v1.1.0 -m "v1.1.0"
git tag -a v1.2.0 -m "v1.2.0"
git tag -l -n1
```
```
v1.1.0          v1.1.0
v1.2.0          v1.2.0
```

An **annotated** tag (`-a`) is its own object in `.git/objects` — with a
message, tagger identity, and timestamp, distinct from a lightweight tag
(just a ref pointing straight at a commit). Releases should always use
annotated tags: `git describe`, changelog generators, and GitHub Releases
all expect the tag object's metadata to exist.

## Generating a changelog straight from tagged history

```bash
git log --pretty=format:"%s" v1.1.0..HEAD
```
```
Add CODEOWNERS
Small fix behind flag (squashed)
```

With Conventional Commits, that same command's output is what a tool like
`release-please` or `semantic-release` parses to (a) compute the next
version, (b) group entries into `Features` / `Bug Fixes` / `BREAKING
CHANGES` sections, and (c) write `CHANGELOG.md` and cut the tag — fully
automated, triggered by every merge to `main`:

```yaml
# .github/workflows/release-please.yml
on:
  push: { branches: [main] }
jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
        with:
          release-type: node
```

This workflow keeps a standing "Release PR" open with the accumulated
changelog and version bump; merging *that* PR is what actually creates the
tag and GitHub Release — a human still approves the release, but computes
nothing.

## Publishing a GitHub Release from a tag

```bash
gh release create v1.2.0 \
  --title "v1.2.0" \
  --notes "$(git log --pretty=format:'- %s' v1.1.0..v1.2.0)" \
  dist/app-linux-x64.tar.gz dist/app-macos-arm64.tar.gz
```

A GitHub Release is a database record (title, notes, prerelease flag)
plus optional binary assets, anchored to a tag — it is not itself a Git
object; deleting a Release does not delete the underlying tag or its
commits.

## Pre-releases and version channels

```bash
git tag -a v2.0.0-rc.1 -m "v2.0.0-rc.1"
gh release create v2.0.0-rc.1 --prerelease --notes "Release candidate for testing"
```

SemVer's pre-release identifiers (`-rc.1`, `-beta.2`, `-alpha`) sort
**before** the final version they precede (`2.0.0-rc.1 < 2.0.0`) under
strict SemVer ordering, and `--prerelease` tells GitHub (and package
managers reading GitHub Releases) not to treat this as "latest stable."

## How It Actually Works

- An annotated tag is a distinct object type (`tag`) in the object
  database, alongside blob/tree/commit — `git cat-file -p v1.2.0` shows
  `object <commit-sha>`, `type commit`, `tag v1.2.0`, tagger line, and the
  message, all hashed together into the tag object's own SHA. A
  lightweight tag has none of this; it's purely an entry in
  `.git/refs/tags/` pointing directly at a commit SHA, which is why `git
  describe` (which needs tagger date and message) silently ignores
  lightweight tags unless you pass `--tags`.
- `git describe` — what most release tooling uses under the hood to
  compute "how far past the last tag" — walks back from `HEAD` through
  first-parent history until it finds a commit reachable from an
  annotated tag, then reports `<tag>-<commits-since>-g<short-sha>`. This
  is exactly the format you see in `go version` output and many
  `--version` flags, and it's why a repo with zero tags makes `git
  describe` fail outright: there's nothing to count "distance since."
- SemVer precedence for pre-release versions is defined field-by-field,
  ASCII/numeric comparison on dot-separated identifiers
  (`rc.1 < rc.2 < rc.10` numerically, not lexically, when every identifier
  is all-digits) — tooling that gets this wrong (naive string sort) will
  incorrectly rank `1.0.0-rc.10` before `1.0.0-rc.2`, a real bug class in
  hand-rolled version-comparison code.
- `release-please`'s "standing PR" pattern works by having the bot
  maintain one branch (`release-please--branches--main`) that it
  force-pushes a rebased changelog/version-bump commit to on every run,
  rather than opening a new PR per merge — this keeps exactly one release
  candidate PR open at a time and is why merging it (a normal PR merge,
  triggering the same workflow) is the signal that actually cuts the tag:
  the workflow distinguishes "a release PR was just merged" from
  "a regular commit landed" by checking the PR's head branch name.

## Exercise

Starting from a fresh scratch repo, make three commits with Conventional
Commit messages: one `fix:`, one `feat:`, and one `feat!:` with a
`BREAKING CHANGE:` footer. By hand, work out what the next version number
would be starting from `v1.0.0`, tag it as an annotated tag, and generate
the changelog text with `git log --pretty=format:"- %s" <prev-tag>..HEAD`.
