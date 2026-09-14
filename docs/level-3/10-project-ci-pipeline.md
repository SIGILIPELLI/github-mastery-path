# 10 · Project — CI Pipeline for a Real Project

This capstone wires a small real project up with tests, a CI workflow,
branch protection, and a tagged release — everything from Level 3 working
together the way it actually would on a real repo.

## Step 1 — a tiny project with a real test

```bash
mkdir -p src tests .github/workflows
cat > src/add.js <<'EOF'
function add(a, b) { return a + b; }
module.exports = add;
EOF
cat > tests/add.test.js <<'EOF'
const add = require('../src/add');
if (add(2,3) !== 5) { console.error("FAIL"); process.exit(1); }
console.log("PASS");
EOF
node tests/add.test.js
```
```
PASS
```

The test genuinely runs and genuinely passes locally before anything
touches GitHub — CI should never be your first signal that a test even
executes correctly.

## Step 2 — the CI workflow

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main]
  pull_request:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: node tests/add.test.js
```

```bash
git add -A
git commit -m "Add project with local test + CI workflow"
git log --oneline
```
```
bb88ad4 Add project with local test + CI workflow
```

## Step 3 — push and protect `main`

```bash
git push -u origin main
```

On GitHub: Settings → Branches → protect `main` with "Require a pull
request," "Require status checks to pass" (select the `test` job by
name, which only appears in the list after the workflow has run at least
once), and "Require branches to be up to date."

## Step 4 — the actual dev loop, using everything from this level

```bash
git switch -c feature/subtract
cat > src/subtract.js <<'EOF'
function subtract(a, b) { return a - b; }
module.exports = subtract;
EOF
git add src/subtract.js
git commit -m "Add subtract function (missing test — intentional)"
git push -u origin feature/subtract
gh pr create --title "Add subtract function" --fill
```

CI runs automatically on the PR. Since there's no test for `subtract`,
review should catch that a merge-blocking gap exists even though the `test`
job technically passes — this is exactly why "CI is green" and "this PR is
ready" are not the same claim, and why code review (Level 2, module 07)
still matters even with strong automation.

## Step 5 — fix, get it merged, and confirm the check ties to the SHA

```bash
cat > tests/subtract.test.js <<'EOF'
const subtract = require('../src/subtract');
if (subtract(5,3) !== 2) { console.error("FAIL"); process.exit(1); }
console.log("PASS");
EOF
git add tests/subtract.test.js
git commit -m "Add missing test for subtract"
git push
```

Update the workflow to run both test files, or use a runner (`node
tests/*.test.js` via a small loop) so new test files are picked up
without editing CI each time:

```yaml
      - run: for f in tests/*.test.js; do node "$f" || exit 1; done
```

```bash
gh pr merge --squash --delete-branch
```

## Step 6 — tag the release once CI is green on `main`

```bash
git fetch origin && git switch main && git pull
git tag -a v0.1.0 -m "Initial release with CI"
```
```bash
git tag
```
```
v0.1.0
```
```bash
git push origin v0.1.0
gh release create v0.1.0 --notes-from-tag
```

## Step 7 — a bug slips through: bisect finds it

Weeks later, `add(2,3)` mysteriously returns `23` instead of `5`. Rather
than guessing, use `git bisect run node tests/add.test.js` between the
last known-good tag and `HEAD` (module 07) — the same test file that CI
already runs is exactly what you feed bisect, since it already encodes
"is this commit correct" as an exit code.

## How It Actually Works

This project is a synthesis, not a new mechanism, but the causal chain is
worth stating explicitly:

- The workflow only appears as a selectable required status check
  (Step 3) because GitHub populates that dropdown from **checks that have
  actually reported at least once** for this repo — branch protection
  can't require a check name it has never seen (module 04 + 05 tie
  together here).
- Every push to the PR branch reruns CI against a **new head SHA**
  (module 03/07 of Level 2), which is why "Require branches to be up to
  date" plus required status checks together guarantee the exact code
  combination that gets merged was the exact combination CI verified —
  not an older version of `main` plus untested new commits.
- The tag created in Step 6 is deliberately the *last* step, after merge
  and after CI confirms green on `main` itself — tagging a commit CI
  hasn't verified defeats the entire point of having CI.

## Exercise

Build this exact pipeline in a real (or throwaway) GitHub repo: a small
project, a CI workflow, branch protection requiring that workflow, one PR
with an intentionally incomplete test that a reviewer would (or should)
catch, a merge, and a tagged release. Then introduce a deliberate
regression a few commits later and use `git bisect run` against your own
test file to find it.
