# 01 · Advanced GitHub Actions (matrix, caching, reusable workflows)

Level 3 covered a single job running once. Real pipelines need to test
across multiple configurations, avoid re-downloading the same
dependencies every run, and share logic across many workflows. This still
requires an actual GitHub-hosted runner to execute — the local scratch
environment can validate the YAML shape and reasoning, not run the jobs.

## Matrix builds: one job definition, many combinations

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
        node: [18, 20, 22]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci && npm test
```

This one job definition expands into **6 independent runs** (2 OSes × 3
Node versions), each getting its own status check, running in parallel.
`matrix.exclude` and `matrix.include` let you carve out or add specific
combinations instead of the full cross-product.

## Failing fast vs. seeing every failure

```yaml
    strategy:
      fail-fast: false
      matrix:
        node: [18, 20, 22]
```

Default is `fail-fast: true` — the first failing matrix leg cancels the
rest, which is faster feedback but hides whether *other* legs would also
have failed. Set `false` when you specifically want to see the full
compatibility picture (e.g. "which Node versions actually break").

## Caching dependencies between runs

```yaml
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
```

Or manually, for anything `setup-*` doesn't cover natively:

```yaml
      - uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: pip-${{ hashFiles('requirements.txt') }}
          restore-keys: pip-
```

The cache key is content-derived (`hashFiles`) so a changed
`requirements.txt` automatically produces a cache miss and a fresh
install, while an unchanged lockfile reuses the exact prior cache —
correctness first, speed second.

## Reusable workflows: sharing a pipeline definition

```yaml
# .github/workflows/reusable-test.yml
on:
  workflow_call:
    inputs:
      node-version:
        type: string
        default: '20'
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci && npm test
```

Called from another workflow, even in a different repo:

```yaml
jobs:
  call-shared-test:
    uses: my-org/shared-workflows/.github/workflows/reusable-test.yml@main
    with:
      node-version: '22'
```

This is different from a **composite action** (a bundle of steps you
`uses:` as a single step inside a job) — reusable workflows operate at the
*job* level and can define multiple jobs with `needs:` between them;
composite actions operate at the *step* level inside one job.

## Concurrency: cancelling stale runs

```yaml
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

Every new push to the same branch/PR cancels any still-running CI for the
previous push on that ref — avoids burning runner minutes testing commits
that are already obsolete.

## Artifacts: passing data between jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
```

Each job is a fresh, isolated VM (Level 3) — artifacts are the mechanism
for handing files from one job's filesystem to another's.

## How It Actually Works

- A matrix job isn't one job that loops — GitHub's workflow engine
  literally instantiates N separate job runs from one YAML definition
  before scheduling, each with its own runner, its own log, its own
  status check name (`test (ubuntu-latest, 20)`) — that's why they run in
  true parallel, bounded only by your plan's concurrent-job limit.
- `actions/cache` stores a compressed tarball of the given path in
  GitHub's cache service, addressed by the exact key string you provide;
  a cache **hit** requires an exact key match (falling back to prefix
  matches via `restore-keys` only for a *partial* restore) — this is why
  `hashFiles('requirements.txt')` in the key is the entire mechanism
  ensuring stale dependencies never get silently reused after a lockfile
  change.
- `workflow_call` reusable workflows execute as genuinely separate job
  runs invoked by the calling workflow's run — inputs/secrets must be
  explicitly passed (`secrets: inherit` or named), because the called
  workflow does not automatically share the caller's execution context;
  this explicit boundary is a deliberate security property, not an
  oversight.
- `cancel-in-progress` works by GitHub tracking runs per `concurrency`
  group string; a new run entering an occupied group sends a cancellation
  signal to the currently running job's process on its VM before
  scheduling itself — this is a scheduler-level action, unrelated to
  anything in the workflow's own steps.

## Exercise

Add a matrix testing your project against three language/runtime
versions with `fail-fast: false`, add dependency caching keyed on your
lockfile's hash, and extract the test job into a separate reusable
workflow called from your main CI file. Push and confirm (via `gh run
list`) that all matrix legs report as independent status checks.
