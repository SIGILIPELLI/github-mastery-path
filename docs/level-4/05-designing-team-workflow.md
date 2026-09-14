# 05 · Designing a Team Git/GitHub Workflow

Everything in Levels 1–4 so far — branching models, PR review, CI,
CODEOWNERS, protection rules — is a set of building blocks. This module
puts them together into one coherent, written-down workflow for an actual
team, and shows the config that enforces each decision.

## Step 1: pick and document the branching model

From module 03: GitHub Flow is the right default for a team shipping a
web service continuously — one `main`, short-lived branches, PR-gated
merges, deploy off `main`. Write it down where new contributors will
actually see it, not just in a wiki:

```markdown
# CONTRIBUTING.md

## Branching
- `main` is always deployable.
- Branch names: `feat/<slug>`, `fix/<slug>`, `chore/<slug>`.
- Branches live < 3 days. If a feature needs longer, land it behind a
  flag in small PRs instead of keeping one branch open.
- Squash-merge to `main`; the PR title becomes the commit message.
```

## Step 2: route review with CODEOWNERS

```bash
mkdir -p .github
cat > .github/CODEOWNERS <<'EOF'
* @platform-team
/packages/api/ @backend-team
/packages/web/ @frontend-team
*.md @docs-team
EOF
git add -A && git commit -m "Add CODEOWNERS"
```

Rules are evaluated **last match wins**, same as `.gitignore` — a PR
touching `/packages/api/handlers.go` matches both `*` and
`/packages/api/`, and the more specific later rule (`@backend-team`) wins.
Combined with branch protection's "require review from Code Owners," this
means a PR touching only `packages/web/` never blocks on the backend
team's availability, and vice versa.

## Step 3: enforce it with branch protection + required checks

```bash
gh api repos/OWNER/REPO/branches/main/protection -X PUT --input - <<'EOF'
{
  "required_status_checks": {"strict": true, "contexts": ["lint", "test", "build"]},
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "require_code_owner_reviews": true,
    "dismiss_stale_reviews": true
  },
  "enforce_admins": true,
  "restrictions": null,
  "required_linear_history": true
}
EOF
```

`required_linear_history: true` is the enforcement mechanism for "no merge
commits on `main`" if the team chose squash-only — GitHub rejects any
merge that would introduce a commit with more than one parent, so the
policy from `CONTRIBUTING.md` is not just a suggestion.
`dismiss_stale_reviews: true` invalidates an approval automatically the
moment new commits land on the PR, closing the loop where someone
approves, the author pushes an unrelated change, and it merges unreviewed.

## Step 4: standardize the PR itself

```markdown
# .github/pull_request_template.md
## What
## Why
## How to test
- [ ] Tests added/updated
- [ ] Docs updated if behavior changed
```

```yaml
# .github/ISSUE_TEMPLATE/bug_report.yml
name: Bug report
body:
  - type: input
    id: repro
    attributes:
      label: Steps to reproduce
    validations:
      required: true
```

Templates aren't just formatting — the checklist items in the PR template
are what a reviewer actually checks against, and structured issue forms
(YAML, not free-text Markdown templates) let automation parse fields
reliably instead of regex-scraping a description.

## Step 5: put it in CI, not just docs

A written policy that CI doesn't check will drift. Encode the mechanical
parts as workflow jobs so a violation fails the PR instead of being caught
in review:

```yaml
name: Policy
on: pull_request
jobs:
  commit-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - uses: wagoid/commitlint-github-action@v6
  no-direct-main-commits:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          if git log --oneline origin/main..HEAD --merges | grep -q .; then
            echo "::error::Merge commits are not allowed; rebase instead."; exit 1
          fi
```

## Putting it together: the decision table

| Decision | Config that enforces it |
|---|---|
| Branch model (GitHub Flow) | `CONTRIBUTING.md` + branch naming convention (documentation only — Git doesn't enforce naming) |
| Review routing | `.github/CODEOWNERS` |
| Mandatory review + CI | branch protection `required_pull_request_reviews`, `required_status_checks` |
| No accidental merge commits | `required_linear_history` |
| Stale approvals don't count | `dismiss_stale_reviews` |
| Consistent PR/issue quality | `pull_request_template.md`, `ISSUE_TEMPLATE/*.yml` |
| Commit message format | `commitlint` CI job |

## How It Actually Works

- CODEOWNERS matching reuses Git's own `.gitignore` pattern engine
  (`fnmatch`-style globs), which is why the "last matching pattern wins"
  rule feels familiar — it's the identical precedence rule `.gitignore`
  uses for negated patterns. GitHub parses the file at the PR's **base**
  branch, not the PR branch, specifically so a malicious PR can't
  redirect its own required reviewers by editing CODEOWNERS in the same
  PR.
- `required_linear_history` is enforced at the ref-update hook on GitHub's
  server, the same choke point that runs branch protection generally: when
  a merge request comes in, GitHub inspects the resulting commit's parent
  count before accepting the ref update, and rejects it if count > 1. This
  is a server-side check with no local Git equivalent — a local `git merge
  --no-ff` command has no way to know the remote will reject it until the
  push happens.
- `dismiss_stale_reviews` is implemented by GitHub watching for new pushes
  to the PR's head ref and comparing the new head SHA against the SHA an
  approval was recorded against; it doesn't re-evaluate the *content* of
  the diff (a whitespace-only fixup dismisses reviews exactly like a
  substantive change would), which is a known limitation teams work around
  with `paths-ignore` triggers or by squashing trivial fixups locally
  before pushing.
- Structured issue forms (`ISSUE_TEMPLATE/*.yml`) get parsed into a JSON
  payload GitHub attaches to the issue body as an HTML comment
  (`<!-- ... -->`) containing the field IDs and values verbatim — this is
  what lets a GitHub Action later parse `github.event.issue.body` for the
  `repro` field's exact answer without fragile text scraping, unlike the
  older Markdown-only issue templates.

## Exercise

Write a `CONTRIBUTING.md` and `.github/CODEOWNERS` for a hypothetical
3-team repo (frontend, backend, infra), then design the exact branch
protection JSON payload (`required_status_checks`, `required_pull_request_reviews`,
`required_linear_history`) that would make every rule in your
`CONTRIBUTING.md` actually enforced rather than just documented.
