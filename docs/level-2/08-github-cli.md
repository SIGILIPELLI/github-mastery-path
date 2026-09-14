# 08 · GitHub CLI (gh)

`gh` is GitHub's official command-line tool — it wraps GitHub's REST/GraphQL
API so you never have to leave the terminal for PRs, issues, releases, or
repo administration.

## Checking install and auth

```bash
gh --version
```
```
gh version 2.96.0 (2026-07-02)
https://github.com/cli/cli/releases/tag/v2.96.0
```

```bash
gh auth status
```
```
github.com
  ✓ Logged in to github.com account SIGILIPELLI (keyring)
  - Active account: true
  - Git operations protocol: https
  - Token: gho_************************************
  - Token scopes: 'gist', 'read:org', 'repo'
```

`gh auth login` (interactive, browser or token flow) is how you set this
up the first time; `gh auth status` is the command to always run first
when a `gh` command mysteriously fails — most failures are scope or login
issues, not the command itself.

## The command map

```bash
gh --help
```
```
CORE COMMANDS
  auth:          Authenticate gh and git with GitHub
  browse:        Open repositories, issues, pull requests, and more in the browser
  codespace:     Connect to and manage codespaces
  discussion:    Work with GitHub Discussions (preview)
  gist:          Manage gists
  issue:         Manage issues
  org:           Manage organizations
  pr:            Manage pull requests
  project:       Work with GitHub Projects.
  release:       Manage releases
  repo:          Manage repositories

GITHUB ACTIONS COMMANDS
  cache:         Manage GitHub Actions caches
  run:           View details about workflow runs
  workflow:      View details about GitHub Actions workflows
```

Every noun (`pr`, `issue`, `repo`, `release`, `run`, `workflow`) maps
directly to a GitHub concept you already know from the web UI — `gh` just
gives you the same actions without a browser round-trip.

## Everyday commands

```bash
gh repo clone owner/name          # clone by owner/name instead of a full URL
gh repo view --web                # open the current repo's page in a browser
gh pr status                      # PRs relevant to you: yours, review-requested, etc.
gh pr checkout 42                 # fetch and check out someone else's PR branch locally
gh issue list --assignee @me
gh run list                       # recent Actions workflow runs
gh run watch                      # tail a running workflow live
gh browse                         # open the current repo (or file/line) in the browser
```

## `gh api`: the escape hatch

Anything the REST or GraphQL API exposes but no dedicated subcommand
covers yet:

```bash
gh api repos/:owner/:repo/branches/main/protection
gh api graphql -f query='{ viewer { login } }'
```

`:owner/:repo` auto-fills from the current directory's git remote — no
need to type the full path when you're already inside the repo.

## Aliases

```bash
gh alias set co 'pr checkout'
```
```
X Could not create alias co: name already taken, use the --clobber flag to overwrite it
```

```bash
gh alias list
```
```
co: pr checkout
```

`co` already ships as a built-in alias for `pr checkout` in this version —
a good example of `gh` itself using its own alias mechanism for common
shortcuts. `--clobber` would overwrite it if you wanted a custom mapping
instead.

## Scripting with `gh` and `jq`

```bash
gh pr list --json number,title,author --jq '.[] | "\(.number): \(.title) (\(.author.login))"'
```

`--json` plus `--jq` turns `gh` into a real scripting tool — pull
structured data for reports, bots, or CI steps instead of scraping
human-formatted terminal output.

## How It Actually Works

- `gh` is a thin, well-typed client over GitHub's REST and GraphQL APIs.
  Every subcommand ultimately becomes an HTTP request with your stored
  OAuth token (`gho_...`, visible truncated in `gh auth status`) in the
  `Authorization` header — `gh api` just lets you make that same kind of
  request manually for anything without a dedicated subcommand yet.
- Credentials are stored via your OS keychain by default (note "(keyring)"
  in the auth status output above) rather than a plaintext config file,
  which is why `gh auth status` can confirm login without ever printing
  the full token.
- `gh` also configures Git's credential helper to delegate HTTPS
  authentication to itself (`git config --get credential.https://github.com.helper`
  shows this after `gh auth login`), which is why plain `git push`/`git
  clone` over HTTPS work seamlessly once you're logged into `gh` — Git
  isn't storing a password anywhere, it's asking `gh` for a token on
  demand each time.
- `:owner/:repo` shorthand resolution works by `gh` reading the current
  directory's `.git/config` for a GitHub-shaped remote URL and parsing the
  owner/repo out of it — outside a git repo, or with a non-GitHub remote,
  you must pass `--repo owner/name` explicitly.

## Exercise

Run `gh auth status` to confirm your login, then use `gh pr list --json
number,title,state --jq '.[] | select(.state=="OPEN")'` against a repo you
have access to, to produce a plain list of open PR titles without opening
a browser.
