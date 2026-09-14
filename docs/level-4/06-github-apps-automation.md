# 06 · GitHub Apps & Automation

Automating GitHub beyond a workflow file — a bot that comments on PRs,
labels issues, or reacts to webhooks in real time — means either an OAuth
App (acts *as a user*) or a GitHub App (acts as its own first-class
identity with fine-grained, installation-scoped permissions). Nearly all
serious automation today (Dependabot, Renovate, CI systems) is built as a
GitHub App, not an OAuth App.

## Why GitHub Apps replaced OAuth Apps for automation

| | OAuth App | GitHub App |
|---|---|---|
| Acts as | the authorizing user | itself, a distinct identity |
| Permissions | whatever scopes the user grants (broad: `repo`, `admin:org`) | fine-grained per-resource (`contents: read`, `issues: write`) |
| Rate limit | shared with the user's personal limit | its own pool per installation |
| Installable per-repo | no (all-or-nothing on the account) | yes — a repo owner picks exactly which repos it can see |
| Auth token lifetime | long-lived, revocable only by the user | short-lived (1 hour), minted per-installation |

A bot built as an OAuth App shows up in commit/PR history as "acting on
behalf of `some-user`" and dies the moment that user's token is revoked or
they leave the org. A GitHub App shows up as its own bot identity
(`my-bot[bot]`) and survives personnel changes entirely.

## Registering a GitHub App (what actually gets created)

Creating an app (via `Settings → Developer settings → GitHub Apps → New`)
produces three artifacts you manage as secrets:

```
App ID:            123456
Private key:       -----BEGIN RSA PRIVATE KEY-----\n...
Webhook secret:    (random string you set)
```

The private key signs a JWT the app uses to authenticate *as the app
itself* (to list its installations, for instance); a separate,
installation-scoped access token — minted by exchanging that JWT — is
what it uses to actually call the REST/GraphQL API against one specific
installation's repos.

## Minting an installation token (real request/response shape)

```bash
# 1. Build a JWT signed with the app's private key (iss=App ID, 10-min exp)
JWT=$(python3 - <<'PY'
import jwt, time
payload = {"iat": int(time.time()) - 60, "exp": int(time.time()) + 600, "iss": "123456"}
print(jwt.encode(payload, open("app-private-key.pem").read(), algorithm="RS256"))
PY
)

# 2. Exchange it for an installation token
curl -s -X POST \
  -H "Authorization: Bearer $JWT" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/app/installations/98765/access_tokens
```
```json
{
  "token": "ghs_16C7e42F292c6912E7710c838347Ae178B4a",
  "expires_at": "2026-09-14T16:12:03Z",
  "permissions": { "contents": "read", "pull_requests": "write" }
}
```

That `ghs_...` token is what goes in the `Authorization: token ghs_...`
header for subsequent API calls, and it stops working after one hour —
short-lived by design, so a leaked token has a tiny blast radius compared
to a personal access token.

## Reacting to webhooks (the actual event flow)

```python
# Flask-style sketch of a GitHub App webhook receiver
@app.route("/webhook", methods=["POST"])
def webhook():
    signature = request.headers["X-Hub-Signature-256"]
    expected = "sha256=" + hmac.new(WEBHOOK_SECRET, request.data, hashlib.sha256).hexdigest()
    if not hmac.compare_digest(signature, expected):
        abort(401)

    event = request.headers["X-GitHub-Event"]
    payload = request.json
    if event == "pull_request" and payload["action"] == "opened":
        post_comment(payload["installation"]["id"], payload["pull_request"]["number"],
                     "Thanks for the PR! Running checks now.")
    return "", 200
```

The `X-Hub-Signature-256` check is not optional in production — without
it, anyone who guesses your webhook URL can forge events. GitHub signs
every webhook body with HMAC-SHA256 using the webhook secret you set at
registration; verifying it is the app's only proof the payload really
came from GitHub.

## The lightweight alternative: `GITHUB_TOKEN` inside Actions

For automation scoped to *your own* repo's workflows, you rarely need a
full GitHub App — the automatic `GITHUB_TOKEN` already acts with an
installation-scoped identity for the duration of a single job:

```yaml
permissions:
  pull-requests: write
jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.addLabels({
              owner: context.repo.owner, repo: context.repo.repo,
              issue_number: context.issue.number, labels: ["needs-review"]
            });
```

Reach for a real GitHub App only when automation needs to run *outside*
a workflow trigger (a long-lived server reacting to webhooks in real
time) or across many installations at once (a SaaS product other repos
install).

## How It Actually Works

- The JWT the app signs with its private key uses **asymmetric** signing
  (RS256) specifically so GitHub can verify the app's identity using only
  the *public* key it already has on file from registration — the app
  never transmits its private key over the network at any point, unlike
  a shared-secret scheme where the verifier would need the same secret
  the signer has.
- Installation tokens are minted per-installation and automatically
  scoped to exactly the repos and permissions the installer approved —
  this scoping is enforced by GitHub's authorization layer checking the
  installation ID embedded in the token against its stored
  repo/permission grant on every API call, not by anything client-side;
  a token minted for installation A simply returns 404 (not 403, to avoid
  leaking repo existence) for a repo under installation B.
- Webhook signature verification must use a constant-time comparison
  (`hmac.compare_digest`, not `==`) because a naive string comparison
  short-circuits on the first mismatched byte, and the tiny timing
  difference between failing at byte 1 versus byte 63 is measurable
  enough over many requests to let an attacker recover the correct
  signature byte-by-byte — the same timing-attack class that motivates
  constant-time comparison in cryptographic code generally.
- The Actions-provided `GITHUB_TOKEN` is itself minted from a hidden
  first-party GitHub App installed on every repo — this is why its
  permissions are configured with the same `permissions:` block syntax
  Apps use, why it can't trigger further workflow runs by design (events
  it creates carry `github-actions[bot]` as actor and are filtered from
  re-triggering `on: push`/`on: pull_request` to prevent infinite
  workflow loops), and why it automatically expires when the job ends
  rather than needing explicit revocation.

## Exercise

Sketch the webhook handler logic (pseudocode is fine) for a bot that:
listens for `issues.opened`, checks whether the issue body matches a
"missing required fields" pattern, and if so posts a comment and adds a
`needs-info` label using an installation token. Write out the exact
`X-GitHub-Event` value and JSON path (`payload["issue"]["body"]`, etc.)
your handler would read for each step.
