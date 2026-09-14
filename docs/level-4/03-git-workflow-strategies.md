# 03 · Git Workflow Strategies (GitFlow, Trunk-Based)

Git doesn't enforce a branching model — GitFlow, trunk-based development,
and GitHub Flow are just conventions about which branches exist, how long
they live, and how they merge. This module builds a small GitFlow release
by hand, then contrasts it with trunk-based development on the same repo.

## GitFlow: long-lived `develop`, short-lived `release/*`

```bash
git init -q -b main
echo v1 > app.txt
git add -A && git commit -m "Initial release"

git branch develop
git switch develop
echo feature1 >> app.txt && git commit -aqm "Add feature 1"

git switch main
git switch -c release/1.1 develop
echo "release notes" > CHANGELOG.md
git add -A && git commit -m "Prepare 1.1"

git switch main
git merge --no-ff release/1.1 -m "Merge release 1.1"
git tag -a v1.1.0 -m "v1.1.0"

git log --oneline --graph --all
```
```
*   b6dfdf4 Merge release 1.1
|\
| * 8739f20 Prepare 1.1
| * 0ff1e35 Add feature 1
|/
* 79a9272 Initial release
```

GitFlow's structure: `main` only ever receives merge commits from
`release/*` or `hotfix/*` branches, so `main`'s history is always
"one commit per shipped release." Feature work happens on branches cut
from `develop`, and `--no-ff` is used deliberately — it forces a merge
commit even when a fast-forward is possible, so the release boundary is
visible in `git log` instead of features blending invisibly into `main`.

## Trunk-based: short-lived branches, squash into `main`

```bash
git switch -c feat/small-fix main
echo fix >> app.txt && git commit -am "Small fix behind flag"

git switch main
git merge --squash feat/small-fix
git commit -m "Small fix behind flag (squashed)"

git log --oneline -3
```
```
5bd6fe0 Small fix behind flag (squashed)
b6dfdf4 Merge release 1.1
8739f20 Prepare 1.1
```

Trunk-based development has one long-lived branch (`main`) and branches
that live hours to a couple of days, merged behind feature flags rather
than left unmerged for weeks. `git merge --squash` collapses the topic
branch's commits into a single new commit on `main` without recording a
merge relationship at all — `git log --graph` on trunk-based history shows
a straight line, not the diamond shapes GitFlow produces, because there's
no merge commit object linking the two parent histories.

## Comparing the two side by side

| | GitFlow | Trunk-based |
|---|---|---|
| Long-lived branches | `main`, `develop` | `main` only |
| Feature branch lifetime | days–weeks | hours–days |
| Merge strategy | `--no-ff` merge commits | squash or rebase |
| Release cadence | batched via `release/*` | continuous, flag-gated |
| Best fit | scheduled/versioned releases (libraries, mobile apps) | continuous deployment (web services) |

GitHub Flow is a middle ground used by most GitHub-hosted projects:
one `main`, short-lived feature branches, merged via pull request as soon
as they're reviewed and CI passes — no `develop`, no `release/*`, and
deploys happen straight off `main`.

## Enforcing the model with branch protection

A workflow choice is only real if the repo enforces it. On GitHub:

```bash
gh api repos/OWNER/REPO/branches/main/protection -X PUT --input - <<'EOF'
{
  "required_status_checks": {"strict": true, "contexts": ["ci"]},
  "enforce_admins": true,
  "required_pull_request_reviews": {"required_approving_review_count": 1},
  "restrictions": null
}
EOF
```

This blocks direct pushes to `main` and force-pushes, requires the `ci`
check to pass on the branch's *current* tip (`strict: true` re-runs the
check if `main` moves while the PR is open), and requires at least one
approving review — regardless of the branching model chosen, this is what
actually stops someone from bypassing it.

## How It Actually Works

- A merge commit (`--no-ff`) is a normal commit object with **two or more
  entries in its `parent` field** instead of one. `git log --graph`
  renders the diamond shape purely by walking those parent pointers; there
  is no separate "merge" bit stored anywhere else. A fast-forward merge
  (what happens without `--no-ff` when the target hasn't diverged) doesn't
  create a commit at all — it just moves the branch ref forward to the
  feature branch's tip, which is why GitFlow insists on `--no-ff`: without
  it, a release with no concurrent hotfix would leave zero trace in
  `main`'s history that a merge ever happened.
- `git merge --squash` computes the merge (a three-way diff against the
  common ancestor, same algorithm as a normal merge) and stages the
  result, but deliberately **does not** record `MERGE_HEAD` or write a
  commit — it leaves that to a plain `git commit`, which then has exactly
  one parent (whatever `main` was pointing at). This is why squashed
  branches never show up as "merged" in `git branch --merged`: from
  Git's graph-reachability perspective, the squash commit shares no
  ancestry with the original topic branch's commits at all, even though
  its content came from them.
- `required_status_checks.strict: true` works by making GitHub re-diff the
  PR's head commit against the *current* `main`, not the `main` the branch
  was created from — internally this is the same merge-base computation
  `git merge-base` performs, so if `main` has moved forward since the
  check last ran, the PR is required to re-run CI against a fresh
  three-way merge to catch integration conflicts the two branches'
  individual histories wouldn't reveal.
- `enforce_admins: true` matters because GitHub's protection rules are
  otherwise bypassable by repo admins by default — protection is enforced
  by the API layer that accepts pushes, not by anything in Git itself, so
  a `git push` still succeeds locally against a mirror or fork; it's
  GitHub's server-side ref-update hook that rejects it against the
  protected branch.

## Exercise

In a scratch repo, build the GitFlow sequence above through the `v1.1.0`
tag, then add a `hotfix/1.1.1` branch cut directly from the `v1.1.0` tag
(not from `develop`), merge it into both `main` and `develop` with
`--no-ff`, and tag the result `v1.1.1`. Use `git log --graph --all` to
confirm the hotfix commit appears in *both* branches' history.
