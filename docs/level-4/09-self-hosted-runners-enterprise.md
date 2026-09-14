# 09 · Self-Hosted Runners & Enterprise GitHub

GitHub-hosted runners are ephemeral VMs GitHub provisions per job. A
self-hosted runner is a long-lived machine (or fleet) you register and
control — needed for hardware GitHub doesn't offer (GPUs, custom
ARM/embedded targets), network access to internal systems, or cost at
scale. Enterprise GitHub layers organization- and enterprise-wide
governance on top of both.

## Registering a self-hosted runner (real steps)

```bash
gh api -X POST repos/OWNER/REPO/actions/runners/registration-token
```
```json
{ "token": "AABBCCDDEEFF...", "expires_at": "2026-09-14T12:00:00Z" }
```

That short-lived token authorizes one runner registration and nothing
else — it is not a credential for the API generally. On the runner
machine:

```bash
mkdir actions-runner && cd actions-runner
curl -o actions-runner.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.319.1/actions-runner-linux-x64-2.319.1.tar.gz
tar xzf actions-runner.tar.gz

./config.sh --url https://github.com/OWNER/REPO \
  --token AABBCCDDEEFF... \
  --labels gpu,cuda-12,on-prem

./run.sh
```

`run.sh` establishes a long-lived outbound HTTPS connection to GitHub and
polls for queued jobs matching this runner's labels — no inbound port
needs to be open on the runner's network, which is what makes self-hosted
runners deployable behind a corporate firewall or NAT.

## Targeting a self-hosted runner from a workflow

```yaml
jobs:
  build-firmware:
    runs-on: [self-hosted, linux, gpu, cuda-12]
    steps:
      - uses: actions/checkout@v4
      - run: make firmware
```

`runs-on` with a list requires the runner to have **every** listed label,
not any — a runner registered with only `[self-hosted, linux]` never
picks up this job.

## The security model self-hosted runners break by default

GitHub-hosted runners are single-use, destroyed after the job. A
self-hosted runner, by default, is **not**: it's the same persistent
filesystem and process across every job, including jobs triggered by a
fork's pull request. This is a real, documented risk — a malicious PR
from a fork can run `runs-on: self-hosted` in its own modified workflow
file (in some trigger configurations) and read whatever secrets or
network access the runner has.

Mitigations, in order of how much they actually close the gap:

```yaml
on:
  pull_request_target:  # rather than pull_request, for fork PRs needing secrets
```

is itself a footgun if misused (it checks out and runs the base repo's
trusted workflow file, but can be tricked into executing untrusted code
if the job then checks out and runs the *fork's* code with secrets
present). The safer patterns:

- Restrict self-hosted runners to `workflow_dispatch` / internal-only
  triggers, never `pull_request` from forks.
- Use **ephemeral** self-hosted runners (`--ephemeral` flag on `config.sh`)
  that deregister and get destroyed after exactly one job, rebuilt fresh
  from an image for the next — eliminating persistent-state leakage
  between jobs entirely.
- Require approval for first-time contributors' workflow runs
  (`Settings → Actions → Fork pull request workflows`).

## Runner groups: enterprise-scale governance

```bash
gh api -X POST orgs/ORG/actions/runner-groups \
  -f name="gpu-fleet" \
  -f visibility="selected" \
  -f selected_repository_ids[]=123456
```

An **enterprise** account (GitHub Enterprise Cloud/Server) can register
runners once at the enterprise level and share them into specific
organizations via runner groups, with per-group policy: which repos may
use the group, whether public-repo (fork) workflows can use it at all
(default: no, for exactly the reason above), and which labels are
permitted.

## Enterprise-level policy that individual repos can't override

| Setting | Scope | What it controls |
|---|---|---|
| Runner groups | Enterprise/Org | which repos can queue jobs on which runner fleets |
| Organization ruleset | Org (applies across repos) | branch/tag protection rules enforced org-wide, can't be weakened per-repo |
| SAML SSO enforcement | Enterprise | membership requires SSO session, independent of 2FA |
| IP allow list | Enterprise | API/Git access restricted to listed CIDR ranges |
| Audit log streaming | Enterprise | every admin action exported to a SIEM (Splunk, S3, Azure) in near-real-time |

Organization **rulesets** (the modern replacement for per-branch
protection rules) are the key scaling mechanism: instead of configuring
branch protection on every repo individually, one ruleset targeting
`~ALL` repos in an org enforces "require signed commits" and "require
CodeQL" everywhere at once, and repo admins cannot weaken it locally.

## How It Actually Works

- The registration token is short-lived (1 hour) specifically to close
  the window where a leaked token could be used to register a rogue
  runner impersonating your infrastructure — it authenticates the
  *registration* handshake only; once `config.sh` completes, the runner
  holds a separate long-lived credential (stored locally, never
  transmitted again) it uses to poll for jobs.
- A self-hosted runner never receives an inbound connection from GitHub
  at all — jobs are delivered over the *same outbound* long-poll
  connection the runner initiated in `run.sh`, using a message queue
  GitHub's Actions backend feeds. This is why self-hosted runners work
  transparently behind restrictive corporate firewalls that block all
  inbound traffic, and also why a runner that loses network connectivity
  simply stops picking up jobs rather than becoming unreachable in a way
  that fails loudly.
- `pull_request_target`'s danger comes from a specific combination:
  it runs with the **base** repo's workflow file and secrets (unlike
  `pull_request`, which uses the fork's workflow file and withholds most
  secrets), but if that trusted workflow then does `actions/checkout` with
  `ref: ${{ github.event.pull_request.head.sha }}` to pull in the fork's
  *code*, the fork's code now executes with the base repo's trusted
  secret context — the vulnerability is the workflow author combining a
  trusted trigger with an untrusted checkout, not a bug in GitHub itself.
- Organization rulesets are enforced at a layer above individual repo
  settings: a ruleset evaluation happens on every ref update alongside
  (and in addition to) any repo-level branch protection, and a repo
  admin's attempt to disable a required check that an org ruleset also
  requires simply has no effect on the org-level requirement — the two
  are separate enforcement points that both have to pass, which is the
  mechanism that makes org-wide policy actually un-bypassable by a single
  repo's admin.

## Exercise

Design the labels and `runs-on` array for three workflow jobs that need,
respectively: (1) an ARM64 build target, (2) GPU access for a model
training step, (3) network access to an internal artifact server behind a
VPN. For each, write the self-hosted runner registration command
(`config.sh ... --labels ...`) that would make exactly the right runner
pick up exactly that job and no others.
