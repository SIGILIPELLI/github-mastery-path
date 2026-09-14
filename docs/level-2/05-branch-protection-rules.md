# 05 · Branch Protection Rules

Branch protection rules are repository settings that restrict what can
happen to a branch directly on GitHub's servers — they can't be simulated
by local Git commands (a local clone has no concept of "protected"), so
this module documents exact settings and their enforced effect.

## Where they live

Settings → Branches → Add branch protection rule (classic), or the newer
Settings → Rules → Rulesets, which apply to branch name patterns
(`main`, `release/*`) across one or many branches at once. A ruleset
example, as a repo-stored config via the API:

```bash
gh api repos/:owner/:repo/rulesets --method POST -f name="main-protection" \
  -f target=branch -f enforcement=active \
  -f 'conditions[ref_name][include][]=refs/heads/main'
```

## The rules that matter

- **Require a pull request before merging** — disables direct pushes to
  the branch entirely; every change must go through a PR, even for repo
  admins if "Do not allow bypassing" is also checked.
- **Require approvals** (e.g. 1 or 2) — a PR's merge button stays disabled
  until that many approving reviews exist and none are stale/dismissed.
- **Dismiss stale reviews on new commits** — any new push to the PR head
  invalidates prior approvals, forcing re-review of the latest code.
- **Require status checks to pass** — names specific CI job(s) (e.g.
  `build`, `test`) that must report success on the exact head commit SHA
  before merge is allowed.
- **Require branches to be up to date before merging** — blocks merging a
  PR whose base has moved on since the PR's branch last incorporated it,
  forcing a rebase/merge-in-latest-main first.
- **Require signed commits** — rejects any commit in the PR that isn't
  GPG/SSH-signed (Level 4 covers signing setup).
- **Require linear history** — blocks merge commits on this branch
  entirely; only squash or rebase merges are permitted.
- **Restrict who can push** — an allowlist of users/teams/apps permitted
  to push directly, useful for release branches only a bot should touch.
- **Require conversation resolution before merging** — every open review
  comment thread must be marked resolved.

## What happens when you try to bypass one locally

Trying to push directly to a protected branch that requires PRs:

```bash
git push origin main
```
```
remote: error: GH006: Protected branch update failed for refs/heads/main.
remote: error: Changes must be made through a pull request.
To github.com:org/repo.git
 ! [remote rejected] main -> main (protected branch hook declined)
error: failed to push some refs to 'github.com:org/repo.git'
```

This message is real GitHub server output (`GH006`) — the push reaches
GitHub, GitHub's pre-receive hook evaluates the ruleset, and rejects the
ref update before it ever touches the branch. Your local repository still
has the commit; only the remote update failed, so `git log` locally shows
it fine.

## Force-push protection specifically

Even without full "require PR" protection, a branch can independently
block force pushes and deletion:

```bash
git push --force origin main
```
```
remote: error: GH003: Sorry, force-pushing to main is not allowed.
```

This is the single most common protection to enable on `main` even on
small/solo repos, since a stray `git push --force` from an old local
branch can otherwise silently discard teammates' commits on the remote.

## Verifying rules via CLI

```bash
gh api repos/:owner/:repo/branches/main/protection | jq .
```

Returns the exact JSON of enforced settings — required checks, review
count, and admin enforcement — useful for auditing multiple repos with a
script instead of clicking through Settings pages one by one.

## How It Actually Works

- Branch protection is enforced as a **pre-receive check on GitHub's
  server**, not a Git feature at all — vanilla Git has no concept of a
  protected ref. When you `git push`, the ref update request reaches
  GitHub's Git server, which runs its protection evaluation (PR-only?
  signed? checks passing? approvals met?) *before* accepting the new ref
  value, and returns a rejection over the Git wire protocol if any rule
  fails — that's the `[remote rejected]` line, which is Git faithfully
  reporting what the server refused, not something your local Git
  invented.
- "Require status checks" ties to a **commit SHA**, not a branch name or
  PR number: GitHub records `{sha, check_name, status}` tuples via the
  Checks API (used by GitHub Actions and third-party CI). Merge is gated
  on the PR's current head SHA having a passing record for every required
  check name — push a new commit and the SHA changes, so all prior
  passing records become irrelevant and checks must rerun.
- "Require branches to be up to date" is enforced by comparing the PR
  head's merge-base against the base branch's current tip; if they differ,
  merging could theoretically combine code that was never actually tested
  together by CI (since CI ran against an older base), so GitHub blocks it
  until you update the branch (merge or rebase main into it) and CI
  reruns against the new combination.
- None of this data is ever transferred by `git clone` or `git fetch` —
  it's queried live from GitHub's API/UI each time, which is why cloning a
  protected repo doesn't tell you anything about its protection rules;
  you'd need `gh api repos/:owner/:repo/branches/main/protection`
  specifically.

## Exercise

On a repo you own, enable "Require a pull request before merging" and
"Require approvals: 1" on `main`. Try pushing a commit directly to `main`
and capture the rejection message. Then open a PR for the same change,
have (or simulate) an approval, and confirm the merge button only becomes
enabled once the approval requirement is satisfied.
