# 09 · GitHub Secrets & Environments

Secrets and Environments are GitHub-server-side features for giving
Actions workflows access to credentials safely — encrypted at rest,
never printed to logs, and (with Environments) gated by approval rules.
There's no local Git equivalent to run; this module documents the real
CLI surface and exact protection semantics.

## Setting a repository secret

```bash
gh secret set NPM_TOKEN --body "sk-abc123..."
```

From `gh secret --help`:

```
Secrets can be set at the repository, or organization level for use in
GitHub Actions, Agents, or Dependabot. User, organization, and repository
secrets can be set for use in GitHub Codespaces. Environment secrets can
be set for use in GitHub Actions.

AVAILABLE COMMANDS
  delete:        Delete secrets
  list:          List secrets
  set:           Create or update secrets
```

```bash
gh secret list
```
```
NAME         UPDATED
NPM_TOKEN    about 1 minute ago
```

Note: `gh secret list` never shows the value, only the name and last
update time — that's not a display limitation, GitHub genuinely cannot
retrieve a secret's plaintext once set; it can only be decrypted inside a
running Actions job.

## Using a secret in a workflow

```yaml
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

If a step accidentally prints the value (`echo $NODE_AUTH_TOKEN`), GitHub
automatically masks the exact substring in the log output, replacing it
with `***` — a scan applied to every line of log output as it's produced.

## Repository, organization, and environment scope

| Scope | Set with | Visible to |
|---|---|---|
| Repository | `gh secret set NAME` | every workflow in that repo |
| Organization | `gh secret set NAME --org myorg` | selected/all repos in the org |
| Environment | `gh secret set NAME --env production` | only jobs targeting that environment |

## Environments: adding approval gates

An **environment** (e.g. `production`) is a named target a job can
declare:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - run: ./deploy.sh
```

Configuring `production` under Settings → Environments lets you require:

- **Required reviewers** — the job pauses (visible as "Waiting" in the
  Actions UI) until a named person/team approves the specific run.
- **Wait timer** — a mandatory delay before the job can start, even with
  approval.
- **Deployment branch restrictions** — only specific branches (e.g. `main`
  or `release/*`) may trigger a deployment to this environment at all.

This is the mechanism behind "someone must click approve before this
touches production" — it's enforced by the Actions runner scheduler
itself refusing to start the job's VM until the gate clears, not by
anything in the workflow YAML.

## Auditing secrets across an org

```bash
gh api orgs/myorg/actions/secrets --jq '.secrets[].name'
```

Useful for a security review: confirm no unused/stale secrets remain
after a service is decommissioned, since forgotten long-lived credentials
are a common real-world source of leaked-token incidents.

## How It Actually Works

- A secret's value is encrypted client-side before it ever leaves your
  machine: `gh secret set` fetches the repository's public key (via the
  API), encrypts the value with `libsodium` sealed-box encryption
  locally, and only the ciphertext is sent to GitHub. GitHub stores the
  ciphertext and holds the matching private key in a hardware-backed
  secrets vault it doesn't expose through any API — this is the actual
  reason "you can set but never read back" isn't a UI restriction, it's a
  cryptographic guarantee.
- At workflow run time, the Actions runner (not the workflow file itself)
  requests the decrypted secret from GitHub's backend over an
  authenticated, short-lived channel scoped to that exact job — this is
  why a compromised runner can only exfiltrate secrets for the *jobs
  it's actually executing*, not the whole repo's secret store at will.
- Log masking works via a substring-replace pass GitHub's log service
  applies to every line before storing/displaying it, checking against
  every secret value known to be in scope for that job — this is also
  why masking can be defeated by trivially transforming a secret first
  (base64-encoding it, reversing it) before printing, which is a known
  limitation, not a bug: never treat masking as a substitute for not
  printing secrets at all.
- Environment approval gates are enforced by the Actions scheduling layer
  itself: a job targeting a protected environment is created in a
  "waiting" state and simply never dispatched to a runner until the
  approval/timer/branch conditions are satisfied — the workflow file has
  no way to bypass this from within its own steps, since the gate exists
  entirely outside the job's execution context.

## Exercise

In a repo you control, create a `staging` and a `production` environment.
Add a required reviewer to `production` only. Add a workflow with two
jobs, each deploying to one environment, and trigger a run — observe that
the `staging` job runs immediately while the `production` job sits
"Waiting" until you approve it in the Actions UI.
