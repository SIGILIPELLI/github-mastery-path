---
description: "Pull Requests (create, review, merge) — A pull request (PR) is GitHub's wrapper around a merge: it packages a branch's commits into a reviewable…"
---

# 03 · Pull Requests (create, review, merge)

A pull request (PR) is GitHub's wrapper around a merge: it packages a
branch's commits into a reviewable, discussable unit before they land in
`main`. The underlying Git operation is exactly the merges and rebases
from earlier modules — GitHub adds the collaboration layer on top.

## The local half: preparing a branch

```bash
git switch -c feature/login-form
echo "<form>login</form>" > login.html
git add login.html
git commit -m "Add login form markup"
git push -u origin feature/login-form
```

`git push -u` sets the upstream tracking branch so future `git push` /
`git pull` on this branch need no arguments — GitHub now has the commits,
but there is no PR yet, just a branch.

## Creating the PR

Via the GitHub CLI (this needs a real GitHub remote and network access —
it cannot be demonstrated in an offline scratch repo, but the invocation
is exact):

```bash
gh pr create --title "Add login form" \
  --body "Adds the initial login form markup." \
  --base main --head feature/login-form
```

Behavior, confirmed from `gh pr create --help`:

```
Create a pull request on GitHub.

Upon success, the URL of the created pull request will be printed.

When the current branch isn't fully pushed to a git remote, a prompt will ask where
to push the branch and offer an option to fork the base repository. Any fork created this
way will only have the default branch of the upstream repository. Use `--head` to
explicitly skip any forking or pushing behavior.
```

Equivalently, on github.com: open the repo, GitHub shows a "Compare &
pull request" banner for any recently-pushed branch, or use the
**Pull requests → New pull request** page and pick base/compare branches
manually.

## What a PR actually contains

A PR is not a copy of your commits — it's a live comparison between two
refs (`base` and `head`). Every new commit you push to `feature/login-form`
automatically appears on the open PR; there's no separate "update the PR"
step.

```bash
echo "<input name='user'>" >> login.html
git commit -am "Add username field"
git push
```

That push alone updates the already-open PR with the new commit and
re-triggers any configured CI checks.

## Reviewing a PR

Reviewers can leave three kinds of feedback:

- **Comment** — feedback with no verdict, just discussion.
- **Approve** — signals the PR is good to merge (required by branch
  protection rules, covered next module).
- **Request changes** — blocks merging until addressed and re-reviewed.

Via CLI:

```bash
gh pr review 42 --approve --body "LGTM, nice tests"
gh pr review 42 --request-changes --body "Please add a null check on line 12"
gh pr diff 42
```

Inline comments (attached to a specific line of the diff) are the most
useful review tool — they show up directly on the line in question and
threaded replies keep discussion scoped to that exact change.

## Merging a PR

GitHub offers three merge strategies, each corresponding to a real Git
operation:

| GitHub button | Git equivalent | Resulting history |
|---|---|---|
| **Create a merge commit** | `git merge --no-ff` | Merge commit with two parents, all original commits kept |
| **Squash and merge** | all PR commits combined into one, then applied on top of base | One clean commit on `main`; original branch commits disappear from `main`'s history |
| **Rebase and merge** | `git rebase` each commit onto base, no merge commit | Linear history, original commits replayed with new hashes |

```bash
gh pr merge 42 --squash --delete-branch
```

Squash is popular for feature branches with messy "wip", "fix typo"
commits — the branch's noisy history never touches `main`, only the final
squashed commit does.

## Draft PRs

Opening a PR before it's ready for review (to get CI running early, or to
share work-in-progress) uses draft mode:

```bash
gh pr create --draft --title "WIP: login form"
gh pr ready 42     # mark ready for review once done
```

Draft PRs cannot be merged until marked ready, which prevents an
accidental early merge.

## How It Actually Works

- A PR is a GitHub-side object, not a Git object — there is nothing in
  `.git/` representing "PR #42." GitHub stores it as metadata (title,
  body, reviews, status) that references two git refs: the base branch
  and the head branch (or a special `refs/pull/42/head` ref GitHub
  maintains server-side for every PR, fetchable with
  `git fetch origin pull/42/head:pr-42` even without the head repo's
  remote configured).
- The "diff" shown on a PR is computed the same way `git diff base...head`
  is computed locally: from the merge base of the two branches, not from
  head's very first commit — this is why a PR only shows *your* changes
  even if `main` has moved on since you branched, as long as your branch
  wasn't rebased onto a stale base.
- **Squash merge** is literally `git merge --squash` plus a manual commit
  under the hood: Git stages the combined diff of all PR commits against
  base but does *not* create a merge commit or record the branch as merged
  in the graph — one ordinary commit gets appended to base's history with
  a single parent, which is why the original per-commit history vanishes
  from `main` (it still exists on the now-orphaned feature branch/ref
  until deleted and garbage collected).
- **Rebase merge** performs the same commit-by-commit replay described in
  the rebase module, server-side, then fast-forwards the base branch —
  each original commit gets a new hash but keeps its individual message
  and diff.
- Required status checks (CI) work by GitHub recording the check result
  against the exact head commit SHA; if you push a new commit, the SHA
  changes and all prior check results become irrelevant to the new SHA,
  which is why pushing always re-runs checks rather than reusing old
  results.

## Exercise

Push a two-commit feature branch to a real (or throwaway) GitHub repo you
control, open a PR with `gh pr create`, then merge it three separate times
in three separate throwaway branches — once with a merge commit, once
squashed, once rebased — and compare the resulting `main` history with
`git log --oneline --graph` for each.
