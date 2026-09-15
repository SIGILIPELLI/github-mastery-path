---
description: "Code Review Etiquette & Workflow — Code review is a social process wrapped around the Git/GitHub mechanics you've already learned. The tools (gh pr…"
---

# 07 · Code Review Etiquette & Workflow

Code review is a social process wrapped around the Git/GitHub mechanics
you've already learned. The tools (`gh pr review`, inline comments) are
simple; doing it well is the actual skill.

## As the author: making a PR easy to review

```bash
gh pr create --title "Add email validation to signup form" \
  --body "$(cat <<'EOF'
## What
Adds regex-based email validation on the signup form's client side.

## Why
Fixes #17 — users were submitting malformed emails that failed silently
server-side with no feedback.

## How to test
1. Go to /signup
2. Enter `not-an-email` and submit
3. Should see inline error instead of a blank redirect
EOF
)"
```

Good PR descriptions answer three questions the diff alone can't: **what**
changed, **why** (link the issue), and **how to verify it**. A 400-line
diff with no description is one of the most common causes of slow, sloppy
reviews.

Keep PRs small and single-purpose. A PR that mixes a refactor with a
feature with a formatting pass is nearly impossible to review carefully —
reviewers either rubber-stamp it or spend hours untangling which change
is which.

## As the reviewer: what to actually check

- **Correctness first** — does the logic do what it claims? Trace at
  least one non-trivial path by hand.
- **Tests** — does new behavior have new tests? Do existing tests still
  make sense given the change?
- **Scope creep** — does the diff match the stated purpose, or did
  unrelated files sneak in?
- **Readability for the next person**, not just "does it work" — naming,
  comments where logic isn't obvious, no leftover debug prints.

```bash
gh pr diff 42
gh pr checkout 42        # pull the PR branch locally to actually run it
```

Pulling a PR down and running it locally (`gh pr checkout`) catches things
a diff view never will — actually exercising the changed code path.

## Leaving comments that help instead of stalling

- Prefix nitpicks so authors can triage instantly: `nit: rename this to
  match the existing convention` vs a blocking concern stated plainly.
- Suggest, don't just criticize — use GitHub's suggested-change blocks so
  the author can apply your exact fix with one click:

  ```suggestion
  if (user == null) {
      return Optional.empty();
  }
  ```
- Ask questions instead of asserting when you're unsure: "Is there a
  reason this doesn't use the existing `validateEmail()` helper?" reads
  very differently from "this should use the existing helper."
- Distinguish **blocking** feedback ("this will break in production
  because...") from **optional** feedback ("could simplify, up to you") —
  don't make an author guess which comments gate the merge.

## Responding to review as the author

- Don't take "Request changes" personally — it's a workflow state, not a
  verdict on you.
- Reply to each thread explicitly (even just "done" or "good point, fixed
  in a1b2c3d") rather than silently pushing a fix and leaving the reviewer
  to guess what changed.
- If you disagree, say so with reasoning, not by ignoring the comment or
  quietly overriding it.

```bash
gh pr comment 42 --body "Fixed the null check in a1b2c3d, thanks for catching that"
```

## Resolving conversations

Once a thread's concern is addressed, mark it resolved so the PR's open
threads reflect only what's still outstanding:

```bash
gh api repos/:owner/:repo/pulls/42/reviews  # inspect review state via API
```

(Marking individual review threads resolved is a GitHub UI/GraphQL action
— there's no dedicated `gh pr` subcommand for it as of this CLI version.)

## How It Actually Works

- Every review comment is stored against a **diff hunk position**, not a
  line number in the file's current state — GitHub records `(commit SHA,
  file path, diff position)`. This is why a comment can appear "outdated"
  after a force-push or rebase changes the diff shape: the position it was
  anchored to no longer exists in the new diff, even if the surrounding
  code is unchanged.
- "Dismiss stale reviews on new commits" (branch protection, previous
  module) exists because an approval is recorded against a specific head
  SHA; GitHub has no way to know a new commit didn't undo what was
  approved, so treating any new SHA as unreviewed by default is the safe
  assumption.
- Suggested-change blocks work by encoding a literal replacement for the
  commented line range; clicking "Commit suggestion" makes GitHub itself
  author a new commit (as the acting user, via the API) that applies that
  diff directly to the PR branch — functionally identical to the author
  making that exact edit and pushing, just performed server-side.
- `gh pr checkout 42` works by fetching GitHub's server-maintained
  `refs/pull/42/head` ref (and `refs/pull/42/merge`, a synthetic
  test-merge commit GitHub keeps up to date) into a local branch — it
  doesn't need the head repository configured as a remote, which is how
  you can check out someone else's fork's PR without ever adding their
  fork as a remote yourself.

## Exercise

Open a PR with a deliberately introduced bug and a missing test. Have a
teammate (or review it yourself the next day with fresh eyes) leave one
blocking comment with a suggested-change block and one non-blocking nit.
Apply the suggestion, push a fix commit, and reply to both threads before
merging.
