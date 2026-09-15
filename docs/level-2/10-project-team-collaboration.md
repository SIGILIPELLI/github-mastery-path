---
description: "Project — Team Collaboration Workflow — This project stitches together everything from Level 2: branching, PRs, reviews, protection rules, issues, and…"
---

# 10 · Project — Team Collaboration Workflow

This project stitches together everything from Level 2: branching,
PRs, reviews, protection rules, issues, and tags — the actual daily loop
of a small team shipping through GitHub.

## Scenario

You're one of two contributors on a small project. Simulate the full loop
end to end using the scratch repos from this level.

## Step 1 — set up the shared repo and protect `main`

```bash
mkdir -p /tmp/team-project && cd /tmp/team-project
git init -q -b main
git config user.email you@example.com
git config user.name You
echo "# Team Project" > README.md
git add . && git commit -qm "Initial commit"
```

On a real GitHub-hosted version of this repo, you'd now enable (Settings →
Branches): require a PR before merging, require 1 approval, require
status checks, dismiss stale approvals on new commits — everything from
module 05.

## Step 2 — file the work as an issue first

```bash
gh issue create --title "Add contact form validation" \
  --body "Users can submit the contact form with an empty email field." \
  --label bug
```

## Step 3 — branch, implement, and open a PR referencing the issue

```bash
git switch -c fix/contact-form-validation
cat > contact.js <<'EOF'
function validate(email) {
  return /\S+@\S+\.\S+/.test(email);
}
module.exports = validate;
EOF
git add contact.js
git commit -m "Add email validation for contact form

Fixes #1"
git push -u origin fix/contact-form-validation
gh pr create --title "Add email validation for contact form" \
  --body "Fixes #1. Adds a regex check before submit." --fill
```

## Step 4 — review, request a change, push a fix

A reviewer notices the regex accepts `a@b.` with no TLD content and
requests a change. The author fixes it and pushes again — this is the
exact moment branch protection's "dismiss stale reviews" would reset the
approval state:

```bash
sed -i '' 's#/\\S+@\\S+\\.\\S+/#/\\S+@\\S+\\.\\S{2,}/#' contact.js
git commit -am "Tighten email regex to require a real TLD"
git push
gh pr review 1 --approve --body "Looks good now"
```

## Step 5 — merge with a strategy that fits the team's convention

Squash merge is a common team default for feature branches with iterative
fix-up commits, so `main`'s history reads as one entry per shipped change:

```bash
gh pr merge 1 --squash --delete-branch
```

## Step 6 — confirm the issue closed and tag a release

```bash
gh issue view 1 --json state --jq .state
```
Expected: `"CLOSED"` — the `Fixes #1` in the commit message plus the merge
triggers GitHub's auto-close.

```bash
git fetch origin
git switch main
git pull
git tag -a v0.1.1 -m "Contact form validation fix"
git push origin v0.1.1
gh release create v0.1.1 --notes-from-tag
```

## Step 7 — verify the whole trail

```bash
git log --oneline --graph -5
gh pr list --state merged --limit 5
gh issue list --state closed --limit 5
```

A healthy team trail shows: one issue, one linked PR, one review cycle
visible in the PR's timeline, a squashed commit on `main` referencing the
issue, and a tag/release marking the shipped fix.

## How It Actually Works

This project doesn't introduce new mechanics — it's worth pausing on how
the pieces you already know **chain together causally**:

- The `Fixes #1` string is inert text until the *merge* event fires; only
  then does GitHub's server-side scan act on it (module 04).
- "Dismiss stale reviews" fires because pushing a new commit changes the
  PR's head SHA, and an approval is recorded against a SHA, not a PR
  number (module 05/07) — so the second push genuinely requires a fresh
  approval, not a formality.
- Squash merge produces one commit on `main` (module 03) with a parent
  that is `main`'s previous tip — a completely linear, easy-to-read
  history — while the original branch's messy commits (including the
  fixup) still exist on the now-deletable `fix/contact-form-validation`
  ref until you (or `--delete-branch`) remove it.
- The tag created afterward (module 06) is independent of all of this —
  it's just a permanent pointer at whatever `main`'s tip happens to be
  the moment you decide "this is a release," which is why tagging is
  always the *last* step, after the merge has landed.

## Exercise

Run this entire sequence yourself end to end on a scratch repo (local is
fine for everything except the actual `gh issue`/`gh pr`/`gh release`
calls, which need a real GitHub remote) and produce a short written
timeline: issue opened → branch created → PR opened → review requested →
fix pushed → approved → merged → issue auto-closed → tagged → released.
