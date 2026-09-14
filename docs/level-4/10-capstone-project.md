# 10 · Capstone Project

This module assembles a complete, small-but-real repo setup using every
piece from Level 4: a sane branch model, a monorepo-aware CI pipeline,
enforced review policy, automated releases, and hardening — built and run
step by step in a scratch repo so you see the actual state after each
addition, not just a description of the end result.

## The scenario

A two-package project (`api`, `web`) that a small team ships continuously.
Requirements: PRs only merge with review + passing CI, releases are
tagged and changelogged automatically from commit messages, and secrets
never land in history.

## Step 1: repo + branch model

```bash
git init -q -b main
mkdir -p packages/api packages/web
echo '{"name":"api"}' > packages/api/package.json
echo '{"name":"web"}' > packages/web/package.json
git add -A && git commit -m "Initial monorepo layout"
```

GitHub Flow (module 03): `main` is always deployable, feature branches
are short-lived, merged via PR.

## Step 2: CI scoped to the package that changed

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      api: ${{ steps.filter.outputs.api }}
      web: ${{ steps.filter.outputs.web }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            api: 'packages/api/**'
            web: 'packages/web/**'
  test-api:
    needs: changes
    if: needs.changes.outputs.api == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm --prefix packages/api ci && npm --prefix packages/api test
  test-web:
    needs: changes
    if: needs.changes.outputs.web == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm --prefix packages/web ci && npm --prefix packages/web test
```

This is module 02's path-scoping (fewer wasted CI minutes) combined with
module 01's matrix/reusable-workflow patterns from Level 3–4.

```bash
mkdir -p .github/workflows
# (workflow file written above)
git add -A && git commit -m "feat: add CI workflow"
git log --oneline -3
```
```
708c658 feat: add CI workflow
615405e Add CODEOWNERS
5bd6fe0 Small fix behind flag (squashed)
```

## Step 3: review routing + branch protection

```bash
cat > .github/CODEOWNERS <<'EOF'
/packages/api/ @backend-team
/packages/web/ @frontend-team
EOF

gh api repos/OWNER/REPO/branches/main/protection -X PUT --input - <<'JSON'
{
  "required_status_checks": {"strict": true, "contexts": ["test-api", "test-web"]},
  "required_pull_request_reviews": {"required_approving_review_count": 1, "require_code_owner_reviews": true},
  "enforce_admins": true,
  "required_linear_history": true,
  "restrictions": null
}
JSON
```

## Step 4: hardening

```bash
gh api repos/OWNER/REPO -X PATCH -f security_and_analysis[secret_scanning][status]=enabled
gh api repos/OWNER/REPO -X PATCH -f security_and_analysis[secret_scanning_push_protection][status]=enabled
gh api repos/OWNER/REPO/branches/main/protection/required_signatures -X POST
```

Module 08's controls: no unreviewed merges, no unsigned commits on
`main`, no leaked secrets.

## Step 5: automated, versioned releases

Every commit follows Conventional Commits so release-please can compute
version bumps without a human deciding:

```bash
git tag -a v1.3.0 -m "v1.3.0"
git log --oneline -5
git tag -l -n1 | tail -3
```
```
708c658 feat: add CI workflow
615405e Add CODEOWNERS
5bd6fe0 Small fix behind flag (squashed)
b6dfdf4 Merge release 1.1
8739f20 Prepare 1.1
v1.1.0          v1.1.0
v1.2.0          v1.2.0
v1.3.0          v1.3.0
```

```yaml
# .github/workflows/release-please.yml
on: { push: { branches: [main] } }
jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
```

`feat: add CI workflow` in the log above is exactly the kind of commit
release-please would read to bump `v1.2.0` → `v1.3.0` automatically the
next time its standing release PR is merged.

## Step 6: scale it — self-hosted runner for a GPU-dependent test

```bash
gh api -X POST repos/OWNER/REPO/actions/runners/registration-token
./config.sh --url https://github.com/OWNER/REPO --token <token> \
  --labels gpu,cuda-12 --ephemeral
```

```yaml
  test-ml:
    needs: changes
    runs-on: [self-hosted, gpu, cuda-12]
    steps:
      - uses: actions/checkout@v4
      - run: pytest packages/api/ml_tests/
```

`--ephemeral` (module 09) so this runner never carries state between PRs
from potentially untrusted branches.

## The finished repo, top to bottom

```
.
├── .github/
│   ├── CODEOWNERS
│   ├── pull_request_template.md
│   └── workflows/
│       ├── ci.yml
│       ├── codeql.yml
│       └── release-please.yml
├── packages/
│   ├── api/
│   └── web/
└── CONTRIBUTING.md
```

Every file in that tree traces to a specific module: `.github/CODEOWNERS`
(05), `ci.yml`'s path filtering (02) and matrix/reusable patterns
(Level 3–4 module 01), `codeql.yml` (08), `release-please.yml` (07),
branch protection JSON (03/05/08), and the self-hosted runner job (09).
None of it is exotic — it's the same dozen primitives (refs, protection
API, workflow triggers, tags) recombined for this team's actual
constraints.

## How It Actually Works

- The whole system has exactly one source of truth Git enforces natively:
  the commit graph and its refs. Everything else — required checks,
  CODEOWNERS routing, signature requirements, runner group membership —
  is a policy GitHub's server layer overlays on top of that graph at the
  moment a ref-update is attempted. This is why `git push --force` to a
  local clone of the same commits always "works" from Git's own
  perspective — rejection only happens when the push reaches GitHub's
  API, which is a deliberate design: Git itself has no opinion about
  organizational policy, by design, so all governance lives in the layer
  above it.
- `paths-filter`-style change detection and release-please's version
  computation both fundamentally reduce to the same primitive:
  `git diff --name-only <base>...<head>` for changed files, and `git log
  <last-tag>..HEAD --pretty=%s` for changed commit messages. Every
  "smart" CI/release tool in this module is a thin, well-tested wrapper
  around those two commands plus API calls to act on the result — nothing
  here requires anything Git doesn't already expose.
- The layered enforcement (branch protection + org ruleset + required
  signatures + required status checks) all evaluates at the *same*
  ref-update choke point on GitHub's backend, each an independent
  predicate that must pass. This is deliberately not short-circuited —
  every check runs and all must pass — which is what makes combining
  them safe: adding a new required check can only make merging stricter,
  never accidentally bypass an existing one.

## Exercise

Using the file tree above as a spec, build this repo for real in a
scratch directory: two packages, the path-filtered CI workflow, a
CODEOWNERS file, and three Conventional-Commit-styled commits (`feat`,
`fix`, `feat!`). Tag a release by hand at each point a real
`release-please` run would have, and write out the resulting version
sequence.
