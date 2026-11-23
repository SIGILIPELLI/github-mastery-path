---
title: "GitHub & Git Cheat Sheet: Quick Reference, Beginner to Advanced"
description: "A free GitHub & Git cheat sheet: 16 quick-reference tables from every level of the GitHub & Git Mastery Path, each linked to its full lesson."
---

# GitHub & Git Cheat Sheet

Quick-reference tables from **16 lessons** of the free GitHub & Git Mastery Path, in course order from beginner to advanced. Each section links to the full lesson with explanations and runnable examples.

## Level 1 · Entry

### [01 · What Is Git &amp; Why Version Control?](level-1/01-what-is-git-why-version-control.md)

*Key vocabulary you'll use constantly*

| Term | Meaning |
|---|---|
| Repository ("repo") | A project's folder, plus all of its tracked history |
| Commit | A saved, labeled snapshot of the project at a point in time |
| Branch | An independent line of development, based on some commit |
| Working directory | The files on disk you're currently editing |
| Staging area (index) | Where you assemble what will go into the *next* commit |
| Remote | A copy of the repository hosted elsewhere (e.g. on GitHub) |
| Clone | A full local copy of a remote repository, history included |

### [02 · Installing Git &amp; Configuration](level-1/02-installing-git-config.md)

*Where config lives*

| Level | Flag | File |
|---|---|---|
| System | `--system` | `/etc/gitconfig` (all users on the machine) |
| Global | `--global` | `~/.gitconfig` (your user account) |
| Local | *(none)* | `.git/config` (one specific repository) |

### [04 · Branching Basics](level-1/04-branching-basics.md)

*Naming conventions*

| Pattern | Used for |
|---|---|
| `feature/short-description` | New functionality |
| `fix/short-description` or `bugfix/...` | Bug fixes |
| `chore/short-description` | Maintenance, config, tooling |
| `release/1.2.0` | Preparing a release |

### [09 · .gitignore Basics](level-1/09-gitignore-basics.md)

*Pattern syntax*

| Pattern | Matches |
|---|---|
| `build/` | A directory named `build`, anywhere in the repo |
| `*.log` | Any file ending in `.log`, anywhere in the repo |
| `/config.json` | `config.json` only at the repo root (leading `/` anchors it) |
| `docs/*.pdf` | PDFs directly inside `docs/`, not subfolders |
| `**/*.tmp` | `.tmp` files at any depth |
| `!important.log` | Un-ignore this specific file, overriding an earlier broader rule |
| `# comment` | Ignored by Git, useful for documenting sections |

## Level 2 · Intermediate

### [01 · Merge Conflicts &amp; Resolution](level-2/01-merge-conflicts-resolution.md)

*Conflicts during rebase vs merge*

| Situation      | Abort command      | Continue command    |
|-----------------|--------------------|----------------------|
| `git merge`     | `git merge --abort`| `git commit`         |
| `git rebase`    | `git rebase --abort`| `git rebase --continue` |
| `git cherry-pick` | `git cherry-pick --abort` | `git cherry-pick --continue` |

### [02 · Rebase vs Merge](level-2/02-rebase-vs-merge.md)

*Choosing between them in practice*

| Situation | Prefer |
|---|---|
| Cleaning up your own messy WIP commits before a PR | rebase (interactive, Level 3) |
| Updating your feature branch with the latest `main` before opening/continuing a PR | rebase (if branch is still yours alone) or merge (if others share it) |
| Bringing a finished PR into `main` | merge (usually via GitHub's merge button) |
| Any branch already shared with teammates | merge |

### [03 · Pull Requests (create, review, merge)](level-2/03-pull-requests.md)

*Merging a PR*

| GitHub button | Git equivalent | Resulting history |
|---|---|---|
| **Create a merge commit** | `git merge --no-ff` | Merge commit with two parents, all original commits kept |
| **Squash and merge** | all PR commits combined into one, then applied on top of base | One clean commit on `main`; original branch commits disappear from `main`'s history |
| **Rebase and merge** | `git rebase` each commit onto base, no merge commit | Linear history, original commits replayed with new hashes |

## Level 3 · Advanced

### [02 · Interactive Rebase &amp; History Rewriting](level-3/02-interactive-rebase-history-rewriting.md)

*Opening the interactive rebase*

| Command | Effect |
|---|---|
| `pick` | keep the commit as-is |
| `reword` | keep the commit, but stop to edit its message |
| `edit` | stop right after applying this commit, so you can amend it |
| `squash` | combine into the *previous* commit, keep both messages |
| `fixup` | combine into the previous commit, discard this message |
| `drop` | remove the commit entirely |

### [06 · Git Hooks](level-3/06-git-hooks.md)

*The hooks that matter most*

| Hook | Runs | Typical use |
|---|---|---|
| `pre-commit` | before a commit is created, after staging | linting, formatting checks, blocking secrets |
| `commit-msg` | after the message is written, before commit finalizes | enforcing a message format (e.g. Conventional Commits) |
| `pre-push` | before `git push` sends anything | running the full test suite, blocking force-pushes to protected local branches |
| `post-checkout` | after `git checkout`/`switch` | reinstalling dependencies if `package-lock.json` changed |
| `post-merge` | after a successful merge/pull | same, for merges |

### [09 · GitHub Secrets &amp; Environments](level-3/09-github-secrets-environments.md)

*Repository, organization, and environment scope*

| Scope | Set with | Visible to |
|---|---|---|
| Repository | `gh secret set NAME` | every workflow in that repo |
| Organization | `gh secret set NAME --org myorg` | selected/all repos in the org |
| Environment | `gh secret set NAME --env production` | only jobs targeting that environment |

## Level 4 · Master

### [03 · Git Workflow Strategies (GitFlow, Trunk-Based)](level-4/03-git-workflow-strategies.md)

*Comparing the two side by side*

| | GitFlow | Trunk-based |
|---|---|---|
| Long-lived branches | `main`, `develop` | `main` only |
| Feature branch lifetime | days–weeks | hours–days |
| Merge strategy | `--no-ff` merge commits | squash or rebase |
| Release cadence | batched via `release/*` | continuous, flag-gated |
| Best fit | scheduled/versioned releases (libraries, mobile apps) | continuous deployment (web services) |

### [05 · Designing a Team Git/GitHub Workflow](level-4/05-designing-team-workflow.md)

*Putting it together: the decision table*

| Decision | Config that enforces it |
|---|---|
| Branch model (GitHub Flow) | `CONTRIBUTING.md` + branch naming convention (documentation only — Git doesn't enforce naming) |
| Review routing | `.github/CODEOWNERS` |
| Mandatory review + CI | branch protection `required_pull_request_reviews`, `required_status_checks` |
| No accidental merge commits | `required_linear_history` |
| Stale approvals don't count | `dismiss_stale_reviews` |
| Consistent PR/issue quality | `pull_request_template.md`, `ISSUE_TEMPLATE/*.yml` |
| Commit message format | `commitlint` CI job |

### [06 · GitHub Apps &amp; Automation](level-4/06-github-apps-automation.md)

*Why GitHub Apps replaced OAuth Apps for automation*

| | OAuth App | GitHub App |
|---|---|---|
| Acts as | the authorizing user | itself, a distinct identity |
| Permissions | whatever scopes the user grants (broad: `repo`, `admin:org`) | fine-grained per-resource (`contents: read`, `issues: write`) |
| Rate limit | shared with the user's personal limit | its own pool per installation |
| Installable per-repo | no (all-or-nothing on the account) | yes — a repo owner picks exactly which repos it can see |
| Auth token lifetime | long-lived, revocable only by the user | short-lived (1 hour), minted per-installation |

### [07 · Release Engineering &amp; Versioning Strategy](level-4/07-release-engineering-versioning.md)

*SemVer: what each commit type maps to*

| Commit type | Version bump | Example |
|---|---|---|
| `fix:` | patch (`1.2.0` → `1.2.1`) | bug fix, no API change |
| `feat:` | minor (`1.2.0` → `1.3.0`) | new backward-compatible capability |
| `feat!:` / `BREAKING CHANGE:` footer | major (`1.2.0` → `2.0.0`) | breaks existing consumers |
| `chore:`, `docs:`, `style:`, `refactor:`, `test:` | no bump | no user-facing change |

### [08 · Security Hardening (CODEOWNERS, signed commits, SAST)](level-4/08-security-hardening.md)

*Putting the controls together*

| Risk | Control |
|---|---|
| Unreviewed merge | CODEOWNERS + `require_code_owner_reviews` |
| Impersonated authorship | commit signing + `required_signatures` |
| Leaked credential | secret scanning + push protection |
| Known code vulnerability pattern | CodeQL as a required check |
| Vulnerable dependency | Dependabot alerts + auto-PRs |
| Bypassing all of the above as an admin | `enforce_admins: true` |

### [09 · Self-Hosted Runners &amp; Enterprise GitHub](level-4/09-self-hosted-runners-enterprise.md)

*Enterprise-level policy that individual repos can't override*

| Setting | Scope | What it controls |
|---|---|---|
| Runner groups | Enterprise/Org | which repos can queue jobs on which runner fleets |
| Organization ruleset | Org (applies across repos) | branch/tag protection rules enforced org-wide, can't be weakened per-repo |
| SAML SSO enforcement | Enterprise | membership requires SSO session, independent of 2FA |
| IP allow list | Enterprise | API/Git access restricted to listed CIDR ranges |
| Audit log streaming | Enterprise | every admin action exported to a SIEM (Splunk, S3, Azure) in near-real-time |

---

Go deeper with the full [GitHub & Git Mastery Path](index.md) curriculum or test yourself with the [interview questions](interview-questions.md).
