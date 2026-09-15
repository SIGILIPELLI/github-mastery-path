---
description: "Security Hardening (CODEOWNERS, signed commits, SAST) — Hardening a repo means reducing three specific risks: someone merging unreviewed code, someone…"
---

# 08 · Security Hardening (CODEOWNERS, signed commits, SAST)

Hardening a repo means reducing three specific risks: someone merging
unreviewed code, someone impersonating another contributor's identity in
history, and a known vulnerability shipping unnoticed. Each has a
concrete, checkable control.

## Required reviews via CODEOWNERS (recap, security angle)

```bash
cat > .github/CODEOWNERS <<'EOF'
* @security-team
/infra/ @security-team
/auth/ @security-team
EOF
```

Paired with branch protection's `require_code_owner_reviews: true`, this
is the single highest-leverage control here: it makes "a human from the
right team looked at this diff" a server-enforced precondition for
merging, not a social norm.

## Signed commits: proving *who* actually wrote a commit

```bash
git config user.signingkey <KEY-ID>
git config commit.gpgsign true
git commit -S -m "Add rate limiting to auth endpoint"
```

Note: this scratch environment has no GPG keyring installed, so the
command above cannot be executed here to show a signature — but the
verification side is what matters day to day, and can be shown on any
signed commit already in a real repo's history:

```bash
git log --show-signature -1
```
```
commit 4a7f2e1... (HEAD -> main)
gpg: Signature made Mon Sep 14 10:02:03 2026 IST
gpg: Good signature from "Jane Dev <jane@example.com>" [ultimate]
Author: Jane Dev <jane@example.com>
Date:   Mon Sep 14 10:02:03 2026 +0530

    Add rate limiting to auth endpoint
```

GitHub shows this as a green "Verified" badge next to the commit — it
independently checks the signature against the public key you've
uploaded to your GitHub account, not against anything the pushing client
claims. `git config commit.gpgsign true` makes signing the default for
every commit instead of remembering `-S` each time; SSH keys can sign
commits too (`gpg.format ssh`), reusing keys many developers already have.

## Enforcing signatures server-side

```bash
gh api repos/OWNER/REPO/branches/main/protection/required_signatures -X POST
```

This rejects any push to `main` containing an unsigned commit, closing
the gap where signing is configured locally but nothing stops an
unsigned commit from a misconfigured machine or a compromised CI runner
from landing anyway.

## Secret scanning and push protection

```bash
gh api repos/OWNER/REPO -X PATCH -f security_and_analysis[secret_scanning][status]=enabled
gh api repos/OWNER/REPO -X PATCH -f security_and_analysis[secret_scanning_push_protection][status]=enabled
```

Secret scanning inspects existing history and new pushes for ~200
recognizable credential patterns (AWS keys, GitHub tokens, Stripe keys,
...); push protection is the *preventive* half — it blocks the push
outright at the moment a matching pattern would enter history, before it
ever becomes something you have to rotate-and-rewrite-history to remove.

## Static analysis: CodeQL as a required check

```yaml
# .github/workflows/codeql.yml
name: CodeQL
on:
  push: { branches: [main] }
  pull_request: { branches: [main] }
  schedule: [{ cron: "0 3 * * 1" }]
jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions: { security-events: write }
    strategy:
      matrix: { language: ["javascript", "python"] }
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
        with: { languages: ${{ matrix.language }} }
      - uses: github/codeql-action/analyze@v3
```

Findings land in the repo's **Security → Code scanning alerts** tab and,
if you add `codeql` to `required_status_checks.contexts` in branch
protection, a PR that introduces a new high-severity finding is blocked
from merging exactly like a failing test would be.

## Dependency vulnerabilities: Dependabot

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule: { interval: "weekly" }
    open-pull-requests-limit: 5
```

Dependabot alerts (from GitHub's Advisory Database) fire on every commit
to any branch, independent of this config; the `dependabot.yml` file
controls the *separate* feature of automatically opening version-bump
PRs, not whether vulnerabilities get detected at all — alerts are on for
every public repo and any private repo with the setting enabled,
regardless of this file's existence.

## Putting the controls together

| Risk | Control |
|---|---|
| Unreviewed merge | CODEOWNERS + `require_code_owner_reviews` |
| Impersonated authorship | commit signing + `required_signatures` |
| Leaked credential | secret scanning + push protection |
| Known code vulnerability pattern | CodeQL as a required check |
| Vulnerable dependency | Dependabot alerts + auto-PRs |
| Bypassing all of the above as an admin | `enforce_admins: true` |

## How It Actually Works

- Commit signing signs the commit **object's content** (tree SHA, parent
  SHA(s), author/committer lines, message) with the signer's private key;
  the signature is embedded as an extra header inside the commit object
  itself (visible via `git cat-file -p`, a `gpgsig` field), not stored
  externally. This means the signature and the content it covers are
  bound at the object level — you cannot alter a commit's message or
  parent without invalidating its signature, because doing so changes the
  commit's SHA and the signature no longer verifies against the new
  content.
- Push protection intercepts at `git push` time, before the ref update is
  accepted: GitHub scans every new blob in the pushed pack for secret
  patterns using the same detectors as historical secret scanning, and if
  one matches, the push is rejected with the offending file/line reported
  back to the client — meaning a caught secret never becomes part of
  `main`'s reachable history at all, versus historical scanning, which
  finds secrets *already* merged and requires a rotate-then-purge
  response (`git filter-repo`, force-push, all consumers re-clone).
- CodeQL works by compiling the codebase into a relational database of
  its abstract syntax tree, control-flow graph, and data-flow edges, then
  running declarative queries (written in QL) against that database — this
  is why it can find genuine taint-flow vulnerabilities (user input
  reaching a SQL query unsanitized across multiple function calls) that
  regex- or AST-pattern-based linters miss, at the cost of a real compile
  step per language per run.
- Dependabot alerts are driven by the GitHub Advisory Database matched
  against your repo's dependency graph, which GitHub builds by parsing
  manifest/lockfiles (`package-lock.json`, `go.sum`, ...) on every push —
  this is a static analysis of declared dependencies, not a runtime scan,
  which is why a vulnerable transitive dependency is caught even if your
  code never actually exercises the vulnerable code path.

## Exercise

Design the full `security_and_analysis` and branch protection payload
(as JSON, matching the `gh api` calls above) that would enforce, for a
single repo: required code owner review, required commit signatures, push
protection, and CodeQL as a required status check. Write out each field
and what specifically it prevents.
