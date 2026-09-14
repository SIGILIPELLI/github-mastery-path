# 09 · Forking Workflow & Contributing to OSS

A **fork** is your own full copy of someone else's repository on GitHub.
The forking workflow is how most open-source contributions happen: you
can't push directly to a project you don't own, so you fork it, work in
your copy, and open a PR asking the original maintainers to pull your
changes in.

## Forking on GitHub, cloning locally

```bash
gh repo fork owner/project --clone
```

This creates `your-username/project` on GitHub (a server-side copy) and
clones it locally in one step, automatically wiring up two remotes.

## The two-remote setup, demonstrated locally

Simulating the same shape with two local bare-ish repos (`upstream-demo`
standing in for the original project, `fork-demo` standing in for your
fork):

```bash
git clone /tmp/upstream-demo /tmp/fork-demo
cd /tmp/fork-demo
git remote rename origin upstream
git remote add origin /tmp/fork-demo-mine
git remote -v
```
```
origin	/tmp/fork-demo-mine (fetch)
origin	/tmp/fork-demo-mine (push)
upstream	/tmp/upstream-demo (fetch)
upstream	/tmp/upstream-demo (push)
```

The convention: `origin` = your fork (where you push your branches),
`upstream` = the original project (where you only ever pull/fetch from).

## Doing the work

```bash
git switch -c feature/readme-tweak
echo "fork addition" >> README.md
git commit -am "Add fork addition"
git push origin feature/readme-tweak
```

Then open a PR **from your fork's branch to the original repo's default
branch**:

```bash
gh pr create --repo owner/project --head yourname:feature/readme-tweak --base main
```

## Keeping your fork in sync with upstream

Meanwhile the real project keeps moving. Pull its new commits into your
local `main` (not your feature branch) regularly:

```bash
git fetch upstream
git switch main
git merge upstream/main
```
```
Updating 528019a..2874cdc
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
```

```bash
git log --oneline --graph --all
```
```
* fa5197d Add fork addition
| * 2874cdc Upstream new commit
|/
* 528019a Initial upstream commit
```

This shows the real shape of forked OSS work: your feature branch
(`fa5197d`) and upstream's new work (`2874cdc`) diverged from the same
commit and haven't been reconciled yet — exactly what you'd rebase your
feature branch onto before finishing your PR, so it applies cleanly
against the project's current `main`.

Push the synced `main` back to your fork too, so your fork's `main` stays
a mirror of upstream's:

```bash
git push origin main
```

## Contribution etiquette specific to OSS

- Read `CONTRIBUTING.md` before opening anything — many projects require
  a specific commit format, a signed CLA, or a discussion/issue first for
  non-trivial changes.
- Keep your first PR small. A 10-line typo fix or a well-scoped bug fix is
  far more likely to get merged (and teaches you the maintainers'
  standards) than an ambitious rewrite.
- Reference an existing issue if one exists (`Fixes #123`), or open one
  first to confirm the maintainers want the change before investing time.
- Expect review latency — OSS maintainers are often volunteers; a PR
  sitting for days or weeks is normal, not a rejection.

## How It Actually Works

- A GitHub "fork" is server-side: GitHub creates a new repository record
  that shares underlying Git object storage with the parent repo (via
  GitHub's internal repository network), which is why forking a huge repo
  is near-instant — it isn't actually copying gigabytes of objects
  immediately, it's creating a new ref namespace pointing into shared
  storage, with copy-on-write for genuinely new objects you push. This
  internal sharing is also what lets GitHub show "compare across forks"
  and serve `refs/pull/N/head` efficiently.
- Locally, a fork is nothing special — it's a Git repository just like
  any other; the `origin`/`upstream` naming is pure convention, not a
  distinct remote type. Git treats every remote identically; what makes a
  "forking workflow" work is entirely how *you* choose to push/pull
  against each one.
- `git fetch upstream` followed by `git merge upstream/main` is exactly
  the fast-forward mechanism from the merging-basics module — it works
  cleanly here because your local `main` had no independent commits of
  its own; all your actual work lived on `feature/readme-tweak`, which is
  precisely why the convention is "never commit directly to your fork's
  `main`."
- When the PR you opened eventually merges into the real upstream, your
  fork's `main` still needs an explicit `git fetch upstream && git merge
  upstream/main` afterward — merging your PR on GitHub does not
  automatically update anything in your fork; forks only sync when you
  tell them to.

## Exercise

Recreate the two-repo setup locally (`upstream-demo` and a clone renamed
to use `origin`/`upstream`). Make a commit on each side independently,
then fetch and merge upstream into your fork's `main`, and rebase your
feature branch onto the updated `main` so it's ready to submit as a clean
PR.
