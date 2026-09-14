# 02 · Monorepo Strategies

A monorepo puts multiple projects — services, libraries, frontends — in one
Git repository instead of one repo each. Git itself has no concept of
"packages"; everything below is built on ordinary trees, sparse-checkout,
and path filters layered on top of a single history.

## Setting up a monorepo layout

```bash
git init -q -b main
mkdir -p packages/api packages/web packages/shared
echo '{"name":"api"}' > packages/api/package.json
echo '{"name":"web"}' > packages/web/package.json
echo '{"name":"shared"}' > packages/shared/package.json
git add -A
git commit -m "Initial monorepo layout"
git log --oneline
```
```
a15c28e Initial monorepo layout
```

One commit, one tree, three subtrees (`packages/api`, `packages/web`,
`packages/shared`). There is nothing structurally different from a
single-project repo — a monorepo is a *convention* about what you put in
the tree, not a different Git object model.

## Sparse-checkout: only materialize what you need

A large monorepo can have hundreds of packages. Nobody working on `api`
needs `web`'s files on disk. `git sparse-checkout` tells Git to populate
the working tree with only selected paths, while the full history is still
fetched.

```bash
git clone --no-checkout https://example.com/monorepo.git sparse-clone
cd sparse-clone
git sparse-checkout init --cone
git sparse-checkout set packages/api packages/shared
git checkout main
find . -not -path './.git*' -type f
```
```
./packages/shared/package.json
./packages/api/package.json
```

`packages/web` was never written to disk. `--cone` mode restricts patterns
to whole directories (fast, index-friendly); non-cone mode accepts full
gitignore-style patterns but is slower on very large trees. Add a package
later with `git sparse-checkout add packages/web` — no re-clone needed.

## Scoping CI to only the packages that changed

The other half of monorepo tooling is *not running every package's tests
on every commit*. GitHub Actions does this with `paths` filters and
`git diff` in a job:

```yaml
name: CI
on:
  pull_request:
    paths:
      - "packages/api/**"
jobs:
  test-api:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm --prefix packages/api ci
      - run: npm --prefix packages/api test
```

For finer-grained control than the `paths:` trigger (which fires the
*workflow*, not individual jobs), compute the changed packages yourself
and fan out:

```bash
git diff --name-only origin/main...HEAD | cut -d/ -f1-2 | sort -u
```
```
packages/api
```

Feed that list into a matrix (`strategy.matrix.package`) so only the
packages actually touched in the PR get built and tested. Tools like Nx,
Turborepo, and Bazel formalize this as a dependency graph so a change to
`packages/shared` correctly re-triggers everything that *depends on* it,
not just packages with literal file changes.

## Splitting history out of a monorepo (or into one)

Two commands exist for the reverse operations — pulling one package's
history out as its own repo, or merging an existing repo in as a
subdirectory while keeping its history:

```bash
git subtree split --prefix=packages/api -b api-only
git log --oneline api-only
```
```
7f3c9a1 Initial monorepo layout
```

`git subtree split` rewrites a synthetic history containing only commits
that touched `packages/api`, rooted so it looks like `api` was always its
own repo. The reverse — importing an existing standalone repo as a
subdirectory of the monorepo while preserving its commit history — is
`git subtree add --prefix=packages/new-lib ../new-lib-repo main`. Both
operations rewrite trees, not blobs: no file content is duplicated: object
sharing between the split repo and the source repo is preserved as long as
blobs are byte-identical.

## How It Actually Works

- Sparse-checkout works entirely through the **index** (`.git/index`),
  which gains a `skip-worktree` bit per entry. Files matching sparse
  patterns are checked out normally; everything else stays as an index
  entry with `skip-worktree` set, so Git knows the object exists (and
  tracks it for diffs against `HEAD`) but deliberately doesn't write it
  to the working directory. This is why sparse repos still show accurate
  `git status` and don't accidentally "lose" files — they're absent from
  disk, not from Git's bookkeeping.
- `--cone` mode matters because non-cone sparse-checkout falls back to
  full `.gitignore`-style pattern matching against *every* path in the
  index on every checkout, which is O(entries × patterns) and gets slow
  past tens of thousands of files. Cone mode restricts patterns to
  recursive directory prefixes, letting Git use a much faster tree-based
  walk that only descends into included directories at all.
- A `paths:` filter on a GitHub Actions trigger is evaluated by GitHub's
  backend *before* a workflow run is even created — it diffs the base and
  head commit's trees restricted to the given prefixes using the same
  tree-diff algorithm `git diff` uses locally, so an unrelated push
  produces zero billed CI minutes, not a run that starts and then exits
  early.
- `git subtree split` walks the full commit graph and, for each commit,
  extracts the sub-tree object at the given prefix, then synthesizes a new
  commit object pointing at that sub-tree with rewritten parent pointers
  (skipping commits where the sub-tree didn't change). Because tree/blob
  objects are content-addressed, a sub-tree that's unchanged between two
  monorepo commits produces the exact same tree SHA in the split history,
  so `git log` on the split branch correctly shows only commits where
  `packages/api` actually changed.

## Exercise

In a scratch repo, build a 3-package monorepo like the one above, make one
commit that touches only `packages/api` and another that touches only
`packages/web`. Run `git subtree split --prefix=packages/api -b api-only`
and confirm with `git log --oneline api-only` that only the API-touching
commit appears in the split history.
