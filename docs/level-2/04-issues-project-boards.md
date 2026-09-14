# 04 · GitHub Issues & Project Boards

Issues and Project boards are GitHub's planning layer — tracking bugs,
feature requests, and work items, and organizing them visually. These are
GitHub-hosted features with no local Git equivalent (nothing lives in
`.git/`), so this module documents the CLI/API surface precisely rather
than simulating output offline.

## Creating and managing issues

```bash
gh issue create --title "Login form does not validate email" \
  --body "Steps to reproduce: ..." \
  --label bug --assignee @me
```

Confirmed from `gh issue create --help`:

```
Create an issue on GitHub.

Adding an issue to projects requires authorization with the `project` scope.
To authorize, run `gh auth refresh -s project`.

The `--assignee` flag supports the following special values:
- `@me`: assign yourself
- `@copilot`: assign Copilot (not supported on GitHub Enterprise Server)
```

Other common operations:

```bash
gh issue list --label bug --state open
gh issue view 17
gh issue comment 17 --body "Confirmed on Chrome 128 too"
gh issue close 17 --comment "Fixed in #42"
gh issue edit 17 --add-label "priority:high" --milestone "v1.1"
```

## Linking issues to commits and PRs

A commit message or PR description containing a closing keyword
(`Fixes #17`, `Closes #17`, `Resolves #17`) auto-closes that issue the
moment the PR merges into the repo's default branch:

```bash
git commit -m "Add email regex validation

Fixes #17"
```

This is purely a GitHub-side text scan of commit messages and PR bodies —
Git itself has no concept of an "issue," it's just a commit message
string.

## Labels, milestones, assignees

- **Labels** — free-text tags (`bug`, `enhancement`, `good first issue`)
  used for filtering and triage; a repo can define custom labels with
  colors and descriptions under Settings → Labels.
- **Milestones** — a due-date/version container (`v1.1`) grouping issues
  and PRs toward a release; progress shows as a completion percentage.
- **Assignees** — up to 10 users/teams responsible for the issue; distinct
  from "reviewers" on a PR (assignees implement, reviewers review).

## Issue templates

Templates standardize what information gets collected, stored as Markdown
or YAML forms under `.github/ISSUE_TEMPLATE/`:

```yaml
# .github/ISSUE_TEMPLATE/bug_report.yml
name: Bug report
description: File a bug report
body:
  - type: input
    id: version
    attributes:
      label: Version
      description: What version are you running?
    validations:
      required: true
  - type: textarea
    id: repro
    attributes:
      label: Steps to reproduce
```

Once this file exists in the default branch, "New issue" on GitHub shows
it as a structured form option instead of a blank text box.

## Project boards (Projects v2)

A Project is a configurable table/board/roadmap view over issues and PRs
across one or more repos. Boards are column-based (`Todo`, `In Progress`,
`Done`); moving a card between columns updates a custom field on the
underlying issue, it doesn't move the issue itself anywhere.

```bash
gh project create --owner myorg --title "Q3 Roadmap"
gh project item-add 3 --owner myorg --url https://github.com/myorg/repo/issues/17
gh project field-list 3 --owner myorg
```

Automations are configurable per project (e.g. "when an issue is closed,
move it to Done") — these live entirely in GitHub's UI/API, there's no
local file representing them.

## How It Actually Works

- Issues and Projects are stored in GitHub's own database, addressed by a
  repo-scoped sequential number (`#17`) that is shared between issues and
  PRs in the same repo — that's why PR numbers and issue numbers never
  collide within one repository (a PR is internally "an issue with a
  branch attached").
- The closing-keyword auto-close behavior is implemented as a webhook-like
  server-side scan: when a PR merges, GitHub parses the PR body and every
  squashed/merged commit message for the keyword pattern
  (`close(s|d)?|fix(es|ed)?|resolve(s|d)?` followed by `#N` or a full
  issue URL) and, if found, calls the same API that a human clicking
  "Close issue" would call. It is timing-dependent on the merge event, not
  on the commit being pushed — pushing to an open PR does not close
  anything yet.
- Project v2 "items" are a layer of indirection: an item wraps a
  reference to an issue/PR *plus* a set of project-scoped custom field
  values (status, priority, iteration). The same underlying issue can be
  an item on multiple different projects simultaneously, each with its
  own independent status — moving a card on Project A's board never
  affects Project B's board even if both track the same issue.
- None of this is fetched by `git clone` — a fresh clone of a repository
  contains zero information about its issues or projects, which is a
  common surprise for people expecting `git log` to somehow reflect issue
  history.

## Exercise

In a repo you control (or a scratch one), create an issue template under
`.github/ISSUE_TEMPLATE/`, open a new issue through it, then create a
commit whose message contains `Fixes #<that issue number>` and merge it
via a PR. Confirm the issue auto-closed and that the closing commit is
linked in the issue's timeline.
