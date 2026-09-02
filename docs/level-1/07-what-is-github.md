# 07 · What Is GitHub? (repos, issues, web UI)

**GitHub** is a website (and platform) that hosts Git repositories and adds
collaboration tooling on top of plain Git: a web UI for browsing code,
issue tracking, pull requests for reviewing changes, project boards,
GitHub Actions for automation, and social features (following users,
starring repos). It's not the only such platform — GitLab and Bitbucket are
close alternatives with mostly-equivalent concepts — but GitHub is the most
widely used, and this course uses it throughout.

## Creating a GitHub account

1. Go to [github.com](https://github.com) and click **Sign up**.
2. Use the same email you configured in `git config --global user.email`
   (module 2) so commits you push are correctly linked to your profile.
3. Choose a username — this becomes part of your profile URL
   (`github.com/yourusername`) and every repo URL you create, so pick
   something you're comfortable using publicly and long-term.
4. The free plan is sufficient for everything in this course, including
   unlimited public *and* private repositories.

## Anatomy of a repository page

Every GitHub repo (e.g. `github.com/octocat/Hello-World`) has the same
layout:

- **Code tab** — browse files at the current branch/commit, view a file's
  history, and see the README rendered below the file list.
- **Issues tab** — a lightweight tracker for bugs, feature requests, and
  tasks (covered in depth in Level 2).
- **Pull requests tab** — proposed changes waiting for review/merge
  (covered in Level 2).
- **Actions tab** — CI/CD workflows that run automatically (covered in
  Level 3).
- **Branch selector** (top-left of the file list) — switch which branch's
  files you're viewing.
- **`<> Code` button** — gives you the clone URL (HTTPS or SSH) and options
  to download a ZIP or open in GitHub Desktop/Codespaces.

## Issues: lightweight tracking

An **issue** is a single tracked item — a bug report, a feature idea, a
question — with a title, a description (rendered as Markdown), labels,
assignees, and a comment thread. Anyone with access to the repo (or, on
public repos, anyone with a GitHub account) can open one.

```markdown
Title: Login button doesn't respond on mobile Safari

## Steps to reproduce
1. Open the site on an iPhone in Safari
2. Tap "Log in"

## Expected
Login modal opens

## Actual
Nothing happens; no console errors
```

Issues can be closed manually, or automatically when a linked pull request
merges (by writing "Closes #12" in the PR description — you'll use this in
Level 2).

## Stars, forks, and watching

- **Star** — bookmark a repo you find interesting; also a rough popularity
  signal.
- **Watch** — subscribe to notifications for activity on a repo.
- **Fork** — create your own full copy of someone else's repository under
  your account, which you can freely modify. This is the standard way to
  contribute to a project you don't have write access to (covered in
  module 10, and in depth in Level 2).

## HTTPS vs. SSH — how you'll authenticate

When you clone or push, GitHub needs to know it's really you. Two common
methods:

- **HTTPS** — clone with a URL like `https://github.com/user/repo.git`.
  Historically used your password, but GitHub disabled password
  authentication for Git operations in 2021; you now authenticate with a
  **Personal Access Token (PAT)** used in place of a password, or by
  signing in through the `gh` CLI or Git Credential Manager, which handle
  the token exchange for you.
- **SSH** — clone with a URL like `git@github.com:user/repo.git`, and
  authenticate using an SSH key pair you generate once and add to your
  GitHub account settings. No token to type on every push.

This course sets up authentication concretely in the next module, when you
create and push your first real repository.

## The web editor and Codespaces

For quick edits, GitHub lets you edit files directly in the browser (press
`.` on any repo page to open a full VS Code-like editor, or click the
pencil icon on a file) — commits made this way go straight to the
repository without needing Git installed locally. **Codespaces** goes
further, giving you a full cloud development environment (a container with
your code already cloned) accessible from the browser. Neither replaces
learning the Git CLI — but they're worth knowing about for quick fixes from
a device without your usual setup.

## Exercise

Create a GitHub account if you don't have one (using the email from your
`git config`). Find a well-known open-source repository (for example,
`https://github.com/torvalds/linux` or any project you use). Open its
Issues tab and read three real issues to see how they're structured. Star
the repository. Note down the repo's clone URL from the `<> Code` button —
you'll use a URL just like it in the next module.
