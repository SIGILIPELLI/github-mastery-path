# 04 · GitHub Actions Basics (a simple CI workflow)

GitHub Actions runs workflows on GitHub's own hosted (or self-hosted)
runners in response to repo events — this is genuinely not something you
can execute on your laptop the way `git` commands run locally: there is
no local Actions runtime to demonstrate against in this offline scratch
environment. What follows is a real, working workflow file plus exactly
what happens when GitHub receives it, so you know precisely what to
expect the first time you push one.

## A minimal workflow

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
      - run: npm ci
      - run: npm test
```

Committing and pushing this file to `.github/workflows/` on any branch is
the entire "installation" step — no separate registration, no dashboard
setup. GitHub watches that exact path.

## Reading the file, section by section

- `on:` — the trigger. Here: every push to `main`, and every pull request
  (against any base, by default). Other common triggers: `schedule` (cron),
  `workflow_dispatch` (manual "Run workflow" button), `release`.
- `jobs:` — one or more independent units of work; by default they run in
  parallel unless you add `needs:` to sequence them.
- `runs-on:` — which machine image executes this job. `ubuntu-latest` is a
  GitHub-hosted VM provisioned fresh for this run and destroyed after.
- `steps:` — executed in order, top to bottom, within the job's VM.
  `uses:` runs a reusable Action (someone else's packaged step, referenced
  by repo and version tag); `run:` executes a raw shell command.

## What happens the moment you push

1. GitHub detects the push matches the `on:` trigger.
2. It provisions a fresh VM matching `runs-on:`.
3. It checks out your repo at the exact commit that triggered the run
   (`actions/checkout@v4` does this — without it, the VM starts empty).
4. Each step runs top to bottom; if any step exits non-zero, the job stops
   and is marked failed (subsequent steps skipped, unless marked
   `if: always()`).
5. The result (success/failure per job) is posted back as a **commit
   status check**, visible directly on the commit and on any PR whose head
   is that commit — this is the exact mechanism "required status checks"
   from the branch protection module reads.

## Viewing runs from the CLI

```bash
gh run list --limit 5
gh run view <run-id> --log
gh workflow list
```

## Local syntax sanity-checking (without a runner)

You cannot execute the workflow locally, but you can at least confirm the
YAML itself parses before pushing, avoiding a wasted round-trip:

```bash
python3 -c "import json,sys; import ast" 2>/dev/null  # placeholder check
cat .github/workflows/ci.yml | python3 -c "
import sys
try:
    import yaml
    yaml.safe_load(sys.stdin)
    print('YAML parses OK')
except ImportError:
    print('PyYAML not installed locally — push and let GitHub validate instead')
"
```

Tools like `actionlint` (a dedicated linter for this exact syntax) catch
many more mistakes — unknown action inputs, invalid expressions — before
you ever push, and are worth installing for any repo with more than a
couple of workflows.

## Environment variables and secrets in a step

```yaml
      - run: echo "Building for $ENVIRONMENT"
        env:
          ENVIRONMENT: production
```

Secrets (API keys, tokens) use `${{ secrets.NAME }}` and are covered fully
in module 09 of this level — never hardcode a real credential directly in
a workflow file, since the file is plain text in the repo.

## How It Actually Works

- Actions triggers are implemented as GitHub server-side webhook-style
  event matching: every push/PR/etc. event is checked against every
  workflow file present on the relevant ref, and any that match get
  queued as a "workflow run." This is why a workflow only reacts to
  events on branches where *that version* of the file already exists —
  adding a new trigger only takes effect from the commit that adds it
  onward, not retroactively.
- Each job gets an **isolated, ephemeral VM** (or container) — nothing
  persists between separate jobs or separate runs, which is exactly why
  `actions/checkout` and dependency installation (`npm ci`) must happen
  every single run: there is no warm state to reuse unless you explicitly
  configure caching (Level 4 covers this).
- `uses: actions/checkout@v4` is a full Action: a versioned, packaged unit
  of code (JavaScript or a Docker container) hosted in its own repository,
  fetched and executed by the runner. Pinning `@v4` (a tag) vs. a commit
  SHA is a genuine security decision — Level 4's security-hardening module
  covers why pinning by SHA is safer for supply-chain reasons.
- The "status check" GitHub shows on a commit is written via the Checks
  API, keyed to that commit's exact SHA — a new push produces a new SHA
  and therefore a fresh, independent check run, never a reused result,
  which ties directly back into why branch protection's "up to date"
  requirement exists.

## Exercise

Add `.github/workflows/ci.yml` (the file above, adapted to your project's
actual test command) to a real repo, push it, and use `gh run watch` to
follow the run live. Deliberately break a test, push again, and observe
the failed status check appear on the commit and on any open PR pointing
at it.
