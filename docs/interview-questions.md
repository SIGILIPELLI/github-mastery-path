---
title: "GitHub & Git Interview Questions and Answers (178 Questions)"
description: "178 GitHub & Git interview questions with answers and explanations, from beginner to advanced, organised by level. Free, from the GitHub & Git Mastery Path."
---

# GitHub & Git Interview Questions and Answers

**178 questions** with the correct answer and a short explanation, organised from beginner (Level 1) to advanced (Level 4). Every question comes from a lesson in the free GitHub & Git Mastery Path, so you can open the lesson to go deeper.

!!! tip "Practise, don't just read"
    Each lesson also ends with an interactive quiz, and the [final quiz](quiz.md) tests all four levels. Try to answer each question below before reading the answer.

## Level 1 · Entry

### [01 · What Is Git &amp; Why Version Control?](level-1/01-what-is-git-why-version-control.md)

**Q1.** What does version control give a team?

**Answer:** A recorded history of changes: who changed what, when and why, with the ability to go back

Collaboration safety net.

**Q2.** What's the difference between Git and GitHub?

**Answer:** Git is the version control tool; GitHub hosts Git repositories and adds collaboration features

Git works without GitHub.

**Q3.** Why is Git called distributed?

**Answer:** Every clone has the full history, so most operations are local

Work offline.

**Q4.** What does a commit represent?

**Answer:** A snapshot of the project with a message, author and parent link(s)

Content-addressed.

### [02 · Installing Git &amp; Configuration](level-1/02-installing-git-config.md)

**Q5.** What do `git config --global user.name/user.email` set?

**Answer:** The identity recorded in your commits

Set once per machine.

**Q6.** What does `git config --list --show-origin` show?

**Answer:** All settings and which file each comes from

Debug config.

**Q7.** Where does global config live?

**Answer:** ~/.gitconfig (or ~/.config/git/config)

Per-user.

**Q8.** What does `git init` create?

**Answer:** A .git directory making the folder a repository

Local.

### [03 · The Basic Workflow (init/add/commit/status/log)](level-1/03-basic-workflow.md)

**Q9.** What does git status --short print? (in a new empty repository, identity configured, default branch main)

```bash
echo hi > a.txt
git add a.txt
echo more > b.txt
git status --short
```

**Answer:** A  a.txt
?? b.txt

A = staged new file; ?? = untracked. Verified by running it with real git.

**Q10.** What does git log print after two commits? (in a new empty repository, identity configured, default branch main)

```bash
echo 1 > f; git add f; git commit -qm 'first'
echo 2 >> f; git commit -qam 'second'
git log --format=%s
```

**Answer:** second
first

Newest first. Verified by running it with real git.

**Q11.** What does git diff show for an unstaged edit vs git diff --staged? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f; git commit -qm c
echo b>>f
git diff --stat | tail -1 | sed 's/^ *//'; git add f; echo staged:; git diff --staged --stat | tail -1 | sed 's/^ *//'
```

**Answer:** 1 file changed, 1 insertion(+)
staged:
1 file changed, 1 insertion(+)

Diff compares working tree to the index; --staged compares the index to HEAD. Verified by running it with real git.

**Q12.** What's the staging area (index) for?

**Answer:** Choosing exactly what goes into the next commit

add then commit.

### [04 · Branching Basics](level-1/04-branching-basics.md)

**Q13.** What does this print? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f; git commit -qm c1
git switch -c feature
echo b>>f; git commit -qam c2
git switch -q main
git branch --format='%(refname:short)' | sort | tr '\n' ' '; git log --format=%s | tr '\n' ' '
```

**Answer:** feature main c1

main still points at c1; feature has c2. Verified by running it with real git.

**Q14.** What is a branch in Git?

**Answer:** A movable pointer to a commit

Cheap.

**Q15.** Which branch are we on after this? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f; git commit -qm c
git branch topic
git switch topic
git branch --show-current
```

**Answer:** topic

switch changes HEAD. Verified by running it with real git.

**Q16.** What does `git switch -c name` do?

**Answer:** Creates a new branch and switches to it

Modern checkout -b.

### [05 · Merging Basics](level-1/05-merging-basics.md)

**Q17.** Is this merge a fast-forward? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f; git commit -qm c1
git switch -qc feat
echo b>>f; git commit -qam c2
git switch -q main
git merge feat | head -1 | sed 's/[0-9a-f]\{7\}\.\.[0-9a-f]\{7\}/X..Y/'
```

**Answer:** Updating X..Y

main had no new commits, so Git just moves the pointer. Verified by running it with real git.

**Q18.** How many parents does the commit created by `git merge --no-ff` have? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f; git commit -qm c1
git switch -qc feat
echo b>>f; git commit -qam c2
git switch -q main
git merge -q --no-ff feat -m 'merge feat'
git cat-file -p HEAD | grep -c '^parent'
```

**Answer:** 2

A merge commit is created even when fast-forward is possible, with two parents. Verified by running it with real git.

**Q19.** What does a merge commit have?

**Answer:** Two parents

Joins histories.

**Q20.** When can Git fast-forward?

**Answer:** When the target branch has no commits the source lacks

Linear history.

### [06 · Remotes (clone/push/pull/fetch)](level-1/06-remotes.md)

**Q21.** What is a remote?

**Answer:** A version of the repository hosted elsewhere (e.g. GitHub)

origin by convention.

**Q22.** What does `git fetch` do?

**Answer:** Downloads new commits without changing your working branch

Safe.

**Q23.** What does `git pull` do?

**Answer:** Fetches and then integrates (merge or rebase) the remote changes

fetch + merge.

**Q24.** What does git remote -v show? (in a new empty repository, identity configured, default branch main)

```bash
git remote add origin https://example.com/me/repo.git
git remote -v
```

**Answer:** origin	https://example.com/me/repo.git (fetch)
origin	https://example.com/me/repo.git (push)

Fetch and push URLs. Verified by running it with real git.

### [07 · What Is GitHub? (repos, issues, web UI)](level-1/07-what-is-github.md)

**Q25.** What does GitHub add on top of Git?

**Answer:** Hosting, pull requests, issues, Actions and access control

Collaboration platform.

**Q26.** What's a repository's default branch commonly called?

**Answer:** main

Configurable.

**Q27.** What's a pull request?

**Answer:** A proposal to merge a branch with discussion and review

Review unit.

**Q28.** Why use SSH keys or tokens with GitHub?

**Answer:** Passwordless secure authentication

Passwords aren't accepted for Git.

### [08 · Creating a GitHub Repo &amp; Pushing](level-1/08-creating-github-repo-pushing.md)

**Q29.** What does `git push -u origin main` do?

**Answer:** Pushes the branch and sets upstream tracking

Later git push works alone.

**Q30.** What happens if you push to a branch whose remote has new commits?

**Answer:** The push is rejected until you integrate the remote changes

Pull first.

**Q31.** What does `git clone` do?

**Answer:** Copies a repository with full history and sets origin

Local copy.

**Q32.** What does `git remote add origin URL` do?

**Answer:** Registers a named remote

Connect local to hosted.

### [09 · .gitignore Basics](level-1/09-gitignore-basics.md)

**Q33.** Which files does git status show as untracked after .gitignore? (in a new empty repository, identity configured, default branch main)

```bash
printf '*.log\nbuild/\n' > .gitignore
mkdir build; touch build/a.o app.log keep.txt
git status --short
```

**Answer:** ?? .gitignore
?? keep.txt

Ignored files don't show. Verified by running it with real git.

**Q34.** What does git check-ignore -v report? (in a new empty repository, identity configured, default branch main)

```bash
printf '*.log\n' > .gitignore
touch x.log
git check-ignore -v x.log | cut -f1
```

**Answer:** .gitignore:1:*.log

Shows which rule matched. Verified by running it with real git.

**Q35.** Does .gitignore remove already-tracked files?

**Answer:** No: untrack them with git rm --cached

Only affects untracked files.

**Q36.** Which should never be committed?

**Answer:** Secrets (.env), dependency folders and build output

Ignore them.

### [10 · Project — Clone, Fork &amp; Ship a Repo](level-1/10-project-clone-fork-ship.md)

**Q37.** What does the clone/fork/ship project practice?

**Answer:** The loop of forking, cloning, branching, committing and opening a PR

OSS workflow.

**Q38.** Why fork instead of clone for open source?

**Answer:** You can't push to the original repo, so you push to your fork and open a PR

Permission model.

**Q39.** What does `upstream` usually refer to?

**Answer:** The original repository you forked from

Sync with it.

**Q40.** How do you keep your fork current?

**Answer:** Fetch upstream and merge/rebase main

git fetch upstream.

## Level 2 · Intermediate

### [01 · Merge Conflicts &amp; Resolution](level-2/01-merge-conflicts-resolution.md)

**Q41.** What do conflict markers look like? (in a new empty repository, identity configured, default branch main)

```bash
printf 'base\n'>f; git add f; git commit -qm base
git switch -qc a; echo A>f; git commit -qam a
git switch -q main; git switch -qc b; echo B>f; git commit -qam b
git merge a >/dev/null 2>&1 || true
cat f
```

**Answer:** &lt;&lt;&lt;&lt;&lt;&lt;&lt; HEAD
B
=======
A
&gt;&gt;&gt;&gt;&gt;&gt;&gt; a

HEAD side first, incoming side last. Verified by running it with real git.

**Q42.** How do you finish a conflicted merge?

**Answer:** Edit the file, git add it and git commit (or git merge --continue)

Resolve then stage.

**Q43.** How do you abort a merge?

**Answer:** git merge --abort

Return to the pre-merge state.

**Q44.** How do you reduce conflicts?

**Answer:** Small frequent merges and clear ownership

Integrate often.

### [02 · Rebase vs Merge](level-2/02-rebase-vs-merge.md)

**Q45.** What does rebasing produce? (in a new empty repository, identity configured, default branch main)

```bash
echo 1 > f; git add f; git commit -qm base
git switch -qc feat; echo f>g; git add g; git commit -qm feat
git switch -q main; echo m>h; git add h; git commit -qm main2
git switch -q feat; git rebase -q main
git log --format=%s | tr '\n' ' '
```

**Answer:** feat main2 base

Feature commits are replayed on top of main. Verified by running it with real git.

**Q46.** What's the golden rule of rebase?

**Answer:** Don't rebase commits others have based work on (shared/public history)

It rewrites commits.

**Q47.** Merge vs rebase history shape?

**Answer:** Merge keeps the branching and adds a merge commit; rebase makes it linear

Trade-offs.

**Q48.** After rebasing a pushed branch, what do you need?

**Answer:** A force push (preferably --force-with-lease)

History changed.

### [03 · Pull Requests (create, review, merge)](level-2/03-pull-requests.md)

**Q49.** What's in a good PR?

**Answer:** A focused change, clear description and linked issue, with tests

Reviewable.

**Q50.** What does a draft PR signal?

**Answer:** Work in progress not ready for review

Early feedback.

**Q51.** What happens on 'Squash and merge'?

**Answer:** All PR commits become one commit on the base branch

Cleaner history.

**Q52.** What do required status checks do?

**Answer:** Block merging until CI passes

Quality gate.

### [04 · GitHub Issues &amp; Project Boards](level-2/04-issues-project-boards.md)

**Q53.** How do issues link to PRs?

**Answer:** Keywords like 'Fixes #12' in the PR description close the issue on merge

Traceability.

**Q54.** What are labels and milestones for?

**Answer:** Categorizing and grouping issues by release/goal

Planning.

**Q55.** What does a Projects board show?

**Answer:** Work items in columns/status, with automation

Kanban.

**Q56.** What's a good issue?

**Answer:** Clear title, steps to reproduce, expected vs actual behavior

Actionable.

### [05 · Branch Protection Rules](level-2/05-branch-protection-rules.md)

**Q57.** What do branch protection rules enforce?

**Answer:** Required reviews, status checks and no force-push on key branches

Server-side.

**Q58.** Why require PRs for main?

**Answer:** Review and CI before changes land

Quality.

**Q59.** What's 'dismiss stale approvals'?

**Answer:** New commits invalidate earlier approvals

Safety.

**Q60.** Can admins bypass rules?

**Answer:** Only if 'include administrators' isn't enabled

Configure carefully.

### [06 · Tags &amp; Releases](level-2/06-tags-releases.md)

**Q61.** What object type does an annotated tag have, versus a lightweight tag? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f; git commit -qm c
git tag -a v1.0.0 -m 'first release'
git tag v1.0.1
git for-each-ref refs/tags --format='%(refname:short) %(objecttype)'
```

**Answer:** v1.0.0 tag
v1.0.1 commit

Annotated tags are their own objects (with a message and tagger); lightweight tags are plain refs to a commit. Verified by running it with real git.

**Q62.** What does git push --tags do?

**Answer:** Pushes local tags to the remote

Tags aren't pushed by default.

**Q63.** What's a GitHub Release built on?

**Answer:** A tag, plus notes and attached assets

Release engineering.

**Q64.** What does semver 2.1.4 mean?

**Answer:** Major 2, minor 1, patch 4

MAJOR.MINOR.PATCH.

### [07 · Code Review Etiquette &amp; Workflow](level-2/07-code-review-etiquette.md)

**Q65.** What makes a good review comment?

**Answer:** Specific, kind and actionable, focused on the code not the person

Etiquette.

**Q66.** What should an author do with feedback?

**Answer:** Respond to each point, push fixes and re-request review

Collaboration.

**Q67.** What's nitpick etiquette?

**Answer:** Mark small style preferences as optional ('nit:')

Don't block on taste.

**Q68.** What should reviewers check first?

**Answer:** Correctness, design and tests before style

Prioritize.

### [08 · GitHub CLI (gh)](level-2/08-github-cli.md)

**Q69.** What does `gh pr create` do?

**Answer:** Opens a pull request from the terminal

Fewer clicks.

**Q70.** What does `gh auth login` do?

**Answer:** Authenticates the CLI with GitHub

Required first.

**Q71.** What does `gh pr checkout 12` do?

**Answer:** Checks out that pull request's branch locally

Test PRs.

**Q72.** What can gh api do?

**Answer:** Call any GitHub REST/GraphQL endpoint with your auth

Scripting.

### [09 · Forking Workflow &amp; Contributing to OSS](level-2/09-forking-workflow-oss.md)

**Q73.** What's the forking workflow?

**Answer:** Fork, clone your fork, branch, push to your fork and open a PR to upstream

OSS standard.

**Q74.** Why add an upstream remote?

**Answer:** To pull updates from the original repository

Keep in sync.

**Q75.** What do CONTRIBUTING.md files tell you?

**Answer:** How to contribute: style, tests and process

Read first.

**Q76.** What's a good first contribution?

**Answer:** A small, well-scoped fix or docs improvement

Build trust.

### [10 · Project — Team Collaboration Workflow](level-2/10-project-team-collaboration.md)

**Q77.** What does the team collaboration project combine?

**Answer:** Branches, PRs, reviews, protection rules, issues and tags

Daily loop.

**Q78.** How do you simulate a team alone?

**Answer:** Use multiple accounts/branches and review your own PRs with checklists

Practice.

**Q79.** What proves the process works?

**Answer:** Merged PRs with passing checks and linked issues

Evidence.

**Q80.** What do you retrospect?

**Answer:** Where the workflow slowed you and what to change

Improve.

## Level 3 · Advanced

### [01 · Git Internals (objects, refs, .git directory)](level-3/01-git-internals.md)

**Q81.** What's a commit's content address derived from? (in a new empty repository, identity configured, default branch main)

```bash
echo hi>f; git add f; git commit -qm c
git cat-file -t HEAD; git cat-file -t HEAD^{tree}; git cat-file -t HEAD:f
```

**Answer:** commit
tree
blob

Objects: blob, tree, commit, tag. Verified by running it with real git.

**Q82.** Same content, same blob? (in a new empty repository, identity configured, default branch main)

```bash
echo same>a; echo same>b; git hash-object a b | uniq | wc -l
```

**Answer:** 1

Content-addressed storage deduplicates. Verified by running it with real git.

**Q83.** What is a tree object?

**Answer:** A directory listing pointing to blobs and subtrees

Snapshot structure.

**Q84.** What is a ref?

**Answer:** A name pointing to a commit (branches, tags, HEAD)

.git/refs.

### [02 · Interactive Rebase &amp; History Rewriting](level-3/02-interactive-rebase-history-rewriting.md)

**Q85.** What does squashing with interactive rebase produce? (in a new empty repository, identity configured, default branch main)

```bash
echo 1 > f; git add f; git commit -qm one
echo 2 >> f; git commit -qam two
echo 3 >> f; git commit -qam three
GIT_SEQUENCE_EDITOR="sed -i.bak '2s/pick/squash/;3s/pick/squash/'" git rebase -qi --root >/dev/null 2>&1
git log --format=%s | wc -l
```

**Answer:** 1

Three commits become one. Verified by running it with real git.

**Q86.** What does `git rebase -i HEAD~3` let you do?

**Answer:** Reorder, squash, edit, reword or drop the last three commits

Rewrites history.

**Q87.** When is it safe to rewrite history?

**Answer:** On local unpublished commits

Avoid public rewrites.

**Q88.** What does `fixup` do?

**Answer:** Melds a commit into the previous one discarding its message

Cleaner history.

### [03 · Cherry-Picking](level-3/03-cherry-picking.md)

**Q89.** What does cherry-pick do? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f; git commit -qm base
git switch -qc other; echo b>g; git add g; git commit -qm 'add g'
git switch -q main
git cherry-pick other >/dev/null
git log --format=%s | tr '\n' ' '; ls | tr '\n' ' '
```

**Answer:** add g base f g

Copies the commit onto the current branch. Verified by running it with real git.

**Q90.** When use cherry-pick?

**Answer:** To apply a specific fix to another branch, e.g. a release branch

Backports.

**Q91.** What's the risk?

**Answer:** Duplicate commits with different hashes can cause confusing merges later

Use sparingly.

**Q92.** What does `-x` add?

**Answer:** A note referencing the original commit

Traceability.

### [04 · GitHub Actions Basics (a simple CI workflow)](level-3/04-github-actions-basics.md)

**Q93.** What are GitHub Actions workflows?

**Answer:** YAML files in .github/workflows triggered by events

CI/CD.

**Q94.** What do `on:` triggers define?

**Answer:** Which events start the workflow (push, pull_request, schedule...)

Event-driven.

**Q95.** What's a runner?

**Answer:** The machine that executes jobs

Hosted or self-hosted.

**Q96.** Why pin actions to a commit SHA?

**Answer:** Tags can be moved; a SHA is immutable (supply-chain safety)

Security.

### [05 · Submodules](level-3/05-submodules.md)

**Q97.** What is a submodule?

**Answer:** A repository embedded at a specific commit inside another

Pinned dependency.

**Q98.** How do you clone including submodules?

**Answer:** git clone --recurse-submodules

Or update --init.

**Q99.** What's a common pitfall?

**Answer:** Forgetting to commit/push the submodule pointer update

Two-step changes.

**Q100.** What are alternatives?

**Answer:** Package managers or subtree

Choose per case.

### [06 · Git Hooks](level-3/06-git-hooks.md)

**Q101.** What does a pre-commit hook that fails do? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f
printf '#!/bin/sh\necho blocked; exit 1\n' > .git/hooks/pre-commit; chmod +x .git/hooks/pre-commit
git commit -qm c 2>&1 | head -1; git rev-list --all --count
```

**Answer:** blocked
0

A non-zero exit aborts the commit. Verified by running it with real git.

**Q102.** Are hooks shared via clone?

**Answer:** No: .git/hooks isn't versioned; share them via tools like core.hooksPath or pre-commit

Team setup.

**Q103.** What's a common pre-commit use?

**Answer:** Linting and formatting checks

Fast feedback.

**Q104.** Which hook runs server-side?

**Answer:** pre-receive/update (on the remote)

Policy enforcement.

### [07 · Git Bisect &amp; Debugging History](level-3/07-git-bisect-debugging-history.md)

**Q105.** Commits c1 to c6 exist and the bug (file b) appeared in one of them. Which commit does bisect report as the first bad one? (in a new empty repository, identity configured, default branch main)

```bash
for i in 1 2 3 4 5 6; do echo $i > v; if [ $i -ge 4 ]; then echo bug > b; fi; git add -A; git commit -qm "c$i"; done
git bisect start >/dev/null
git bisect bad HEAD >/dev/null
git bisect good HEAD~5 >/dev/null
git bisect run sh -c '! test -f b' >/dev/null
git log --format=%s -1 refs/bisect/bad
git bisect reset >/dev/null 2>&1
```

**Answer:** c4

Binary search over history for the first bad commit (the test succeeds while file b is absent). Verified by running it with real git.

**Q106.** What does bisect need?

**Answer:** A known good commit and a known bad commit (and a test)

Automate with run.

**Q107.** Complexity of bisect?

**Answer:** O(log n) steps

Binary search.

**Q108.** What makes bisect effective?

**Answer:** Small commits that each build and test

Hygiene.

### [08 · Reflog &amp; Recovering Lost Work](level-3/08-reflog-recovering-lost-work.md)

**Q109.** How do you recover a commit after a hard reset? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f; git commit -qm one
echo b>>f; git commit -qam two
git reset -q --hard HEAD~1
git reflog --format=%gs | head -1 | cut -d: -f1
git reset -q --hard HEAD@{1}
git log --format=%s | head -1
```

**Answer:** reset
two

The reflog records HEAD movements. Verified by running it with real git.

**Q110.** What does reflog record?

**Answer:** Where HEAD and branch tips pointed, locally, for about 90 days by default

Local safety net.

**Q111.** Is a commit lost after reset --hard?

**Answer:** No: it remains in the object database and reflog until garbage collected

Recoverable.

**Q112.** What shows dangling commits?

**Answer:** git fsck --lost-found

Last resort.

### [09 · GitHub Secrets &amp; Environments](level-3/09-github-secrets-environments.md)

**Q113.** Where should CI secrets be stored?

**Answer:** GitHub Actions secrets (repo/environment), never in the repo

Masked in logs.

**Q114.** What do environments add?

**Answer:** Protection rules like required reviewers and environment-scoped secrets

Deploy gates.

**Q115.** Why not print secrets in logs?

**Answer:** Masking can be bypassed and logs are widely visible

Avoid echo.

**Q116.** What's OIDC for cloud deploys?

**Answer:** Short-lived tokens without storing long-lived cloud keys

Keyless.

### [10 · Project — CI Pipeline for a Real Project](level-3/10-project-ci-pipeline.md)

**Q117.** What does the CI pipeline project do?

**Answer:** Builds, tests and gates merges with GitHub Actions

Level 3 project.

**Q118.** How cache dependencies?

**Answer:** actions/cache or setup-* cache options keyed on lockfiles

Speed.

**Q119.** What status check blocks merge?

**Answer:** A required workflow job passing

Branch protection.

**Q120.** How do you debug a failing workflow?

**Answer:** Read logs, rerun with debug logging and reproduce locally (act)

Iterate.

## Level 4 · Master

### [01 · Advanced GitHub Actions (matrix, caching, reusable workflows)](level-4/01-advanced-github-actions.md)

**Q121.** What do reusable workflows do?

**Answer:** Share a workflow across repos via workflow_call

DRY CI.

**Q122.** What are matrix builds?

**Answer:** Running a job across combinations like OS and versions

Coverage.

**Q123.** What do concurrency groups do?

**Answer:** Cancel or queue overlapping runs

Avoid wasted/conflicting runs.

**Q124.** Why least-privilege GITHUB_TOKEN permissions?

**Answer:** Limit damage if a workflow is compromised

permissions: contents: read.

### [02 · Monorepo Strategies](level-4/02-monorepo-strategies.md)

**Q125.** What is a monorepo trade-off?

**Answer:** Atomic cross-project changes and shared tooling versus scale and CI complexity

Choose deliberately.

**Q126.** How avoid running all CI for every change?

**Answer:** Path filters and affected-project detection

Efficiency.

**Q127.** What helps ownership in a monorepo?

**Answer:** CODEOWNERS files

Review routing.

**Q128.** What tools help?

**Answer:** Bazel/Nx/Turborepo for build graphs and caching

At scale.

### [03 · Git Workflow Strategies (GitFlow, Trunk-Based)](level-4/03-git-workflow-strategies.md)

**Q129.** What's trunk-based development?

**Answer:** Short-lived branches merged to main frequently, with feature flags

Continuous integration.

**Q130.** What's GitFlow suited to?

**Answer:** Versioned releases with long support windows

Heavier.

**Q131.** What's GitHub Flow?

**Answer:** Branch from main, PR, review, merge and deploy

Simple.

**Q132.** How choose a workflow?

**Answer:** Match release cadence, team size and risk tolerance

Fit.

### [04 · Large Repo Performance (sparse checkout, partial clone)](level-4/04-large-repo-performance.md)

**Q133.** Why are big repos slow?

**Answer:** Large history and many files make clone/status heavy

Mitigate.

**Q134.** What does a shallow clone do?

**Answer:** Fetches limited history (--depth)

Faster CI.

**Q135.** What does sparse-checkout do?

**Answer:** Checks out only selected directories

Large monorepos.

**Q136.** What's partial clone?

**Answer:** Fetches blobs on demand (--filter=blob:none)

Saves bandwidth.

### [05 · Designing a Team Git/GitHub Workflow](level-4/05-designing-team-workflow.md)

**Q137.** What should a team workflow doc define?

**Answer:** Branching, review rules, CI gates, release process and ownership

Shared expectations.

**Q138.** How reduce review bottlenecks?

**Answer:** Small PRs, CODEOWNERS and clear SLAs

Flow.

**Q139.** How measure workflow health?

**Answer:** PR cycle time, review time and failure rate

Metrics.

**Q140.** What's a good merge policy?

**Answer:** Consistent (e.g. squash) with protected main

Predictability.

### [06 · GitHub Apps &amp; Automation](level-4/06-github-apps-automation.md)

**Q141.** What can GitHub Apps do?

**Answer:** Act as an app with fine-grained permissions and webhooks

Better than PATs.

**Q142.** Why prefer Apps over personal tokens?

**Answer:** Scoped permissions, installable per repo and not tied to a person

Security.

**Q143.** What's a webhook?

**Answer:** An HTTP callback GitHub sends on events

Automation.

**Q144.** What do Probot-style bots automate?

**Answer:** Labeling, triage and policy checks

Workflow help.

### [07 · Release Engineering &amp; Versioning Strategy](level-4/07-release-engineering-versioning.md)

**Q145.** What does semantic-release or release-please do?

**Answer:** Automate versioning and changelogs from conventional commits

Release engineering.

**Q146.** What's a conventional commit?

**Answer:** A structured message like feat:, fix: enabling automation

Parseable.

**Q147.** What's a release branch for?

**Answer:** Stabilizing and patching a version while main moves on

Backports.

**Q148.** Why sign tags/commits?

**Answer:** Verify authenticity of releases

Supply chain.

### [08 · Security Hardening (CODEOWNERS, signed commits, SAST)](level-4/08-security-hardening.md)

**Q149.** What hardens a GitHub org?

**Answer:** 2FA/SSO, least privilege, branch protection and secret scanning

Defense.

**Q150.** What does secret scanning do?

**Answer:** Detects committed credentials and can block pushes

Push protection.

**Q151.** What does Dependabot do?

**Answer:** Opens PRs to update vulnerable or outdated dependencies

Supply chain.

**Q152.** What does CodeQL provide?

**Answer:** Static analysis to find vulnerabilities in code

Code scanning.

### [09 · Self-Hosted Runners &amp; Enterprise GitHub](level-4/09-self-hosted-runners-enterprise.md)

**Q153.** Why self-hosted runners?

**Answer:** Special hardware, private network access or cost at scale

You maintain them.

**Q154.** What's the security risk?

**Answer:** Running untrusted PR code on persistent machines

Use ephemeral runners.

**Q155.** How scale runners?

**Answer:** Autoscaling ephemeral runners (e.g. Kubernetes controller)

Clean per job.

**Q156.** What isolates jobs best?

**Answer:** Ephemeral containers/VMs destroyed after each job

No leftovers.

### [10 · Capstone Project](level-4/10-capstone-project.md)

**Q157.** What does the capstone combine?

**Answer:** Workflow design, CI/CD, security hardening and release automation on GitHub

Team-grade repo.

**Q158.** How prove it's production-ready?

**Answer:** Protected branches, required checks, secret scanning and a tested release

Evidence.

**Q159.** What documentation helps contributors?

**Answer:** README, CONTRIBUTING, CODEOWNERS and templates

Onboarding.

**Q160.** What sustains the repo?

**Answer:** Automation, dependency updates and clear ownership

Maintenance.

## Mixed questions across all levels

**Q161.** What distinguishes Git from GitHub?

**Answer:** Git is the distributed version control tool; GitHub hosts repositories and adds collaboration features

Git works fully offline.

**Q162.** What does git status --short show? (in a new empty repository, identity configured, default branch main)

```bash
echo hi > a.txt
git add a.txt
echo more > b.txt
git status --short
```

**Answer:** A  a.txt
?? b.txt

A = staged new file; ?? = untracked. Verified by running it with real git.

**Q163.** After this, which commit does main point to and what does feature have? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f; git commit -qm c1
git switch -qc feature
echo b>>f; git commit -qam c2
git switch -q main
git log --format=%s | paste -sd' ' -; git log feature --format=%s | paste -sd' ' -
```

**Answer:** c1
c2 c1

A branch is just a pointer; main still points at c1. Verified by running it with real git.

**Q164.** How many parents does the commit made by `git merge --no-ff` have? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f; git commit -qm c1
git switch -qc feat
echo b>>f; git commit -qam c2
git switch -q main
git merge -q --no-ff feat -m 'merge feat'
git cat-file -p HEAD | grep -c '^parent'
```

**Answer:** 2

Even when fast-forward is possible, --no-ff creates a merge commit with two parents. Verified by running it with real git.

**Q165.** What does the conflicted file contain? (in a new empty repository, identity configured, default branch main)

```bash
printf 'base\n'>f; git add f; git commit -qm base
git switch -qc a; echo A>f; git commit -qam a
git switch -q main; git switch -qc b; echo B>f; git commit -qam b
git merge a >/dev/null 2>&1 || true
cat f
```

**Answer:** &lt;&lt;&lt;&lt;&lt;&lt;&lt; HEAD
B
=======
A
&gt;&gt;&gt;&gt;&gt;&gt;&gt; a

The current branch (HEAD) comes first, the incoming branch last. Verified by running it with real git.

**Q166.** What does the history look like after rebasing feat onto main? (in a new empty repository, identity configured, default branch main)

```bash
echo 1 > f; git add f; git commit -qm base
git switch -qc feat; echo f>g; git add g; git commit -qm feat
git switch -q main; echo m>h; git add h; git commit -qm main2
git switch -q feat; git rebase -q main
git log --format=%s | tr '\n' ' '
```

**Answer:** feat main2 base

Rebase replays feat's commits on top of main, producing linear history. Verified by running it with real git.

**Q167.** Which commit does cherry-pick copy? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f; git commit -qm base
git switch -qc other; echo b>g; git add g; git commit -qm 'add g'
git switch -q main
git cherry-pick other >/dev/null
git log --format=%s | tr '\n' ' '
```

**Answer:** add g base

Cherry-pick applies that commit's change as a new commit on the current branch. Verified by running it with real git.

**Q168.** How do you get a commit back after `git reset --hard HEAD~1`? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f; git commit -qm one
echo b>>f; git commit -qam two
git reset -q --hard HEAD~1
git reset -q --hard HEAD@{1}
git log --format=%s | head -1
```

**Answer:** two

The reflog records where HEAD pointed, so the commit is still reachable. Verified by running it with real git.

**Q169.** What happens to a commit when a pre-commit hook exits non-zero? (in a new empty repository, identity configured, default branch main)

```bash
echo a>f; git add f
printf '#!/bin/sh\necho blocked; exit 1\n' > .git/hooks/pre-commit; chmod +x .git/hooks/pre-commit
git commit -qm c 2>&1 | head -1; git rev-list --all --count
```

**Answer:** blocked
0

A failing hook aborts the commit. Verified by running it with real git.

**Q170.** Which commit does bisect report as the first bad one? (in a new empty repository, identity configured, default branch main)

```bash
for i in 1 2 3 4 5 6; do echo $i > v; if [ $i -ge 4 ]; then echo bug > b; fi; git add -A; git commit -qm "c$i"; done
git bisect start >/dev/null
git bisect bad HEAD >/dev/null
git bisect good HEAD~5 >/dev/null
git bisect run sh -c '! test -f b' >/dev/null
git log --format=%s -1 refs/bisect/bad
git bisect reset >/dev/null 2>&1
```

**Answer:** c4

Binary search over history, O(log n). Verified by running it with real git.

**Q171.** `git push` is rejected because the remote has new commits. What do you do?

**Answer:** Fetch and integrate (merge or rebase) the remote changes, then push

Never force-push shared branches casually.

**Q172.** You rebased a branch you already pushed. What push do you need?

**Answer:** A force push, preferably --force-with-lease, and only on your own branch

History was rewritten.

**Q173.** A committed .env file leaked a key. What must you do besides deleting the file?

**Answer:** Revoke/rotate the key; Git history still contains it

Treat it as compromised.

**Q174.** Main must never receive unreviewed or failing code. What do you configure?

**Answer:** Branch protection: required reviews, required status checks and no direct pushes

Server-side enforcement.

**Q175.** A workflow deploys from forks' PRs with secrets and an attacker steals them. What went wrong?

**Answer:** Secrets were exposed to untrusted code (pull_request_target/misconfig); restrict secrets and permissions

Least privilege for tokens.

**Q176.** Why pin third-party actions to a commit SHA?

**Answer:** Tags can be moved to malicious code; a SHA is immutable

Supply-chain safety.

**Q177.** A monorepo's CI runs everything for every change. What helps?

**Answer:** Path filters/affected-project detection and caching

Scale CI.

**Q178.** An old release needs a security fix while main has moved on. What do you do?

**Answer:** Fix on a release branch (or cherry-pick the fix), tag a patch release

Backport.

## Practical coding questions

### Resolve a merge conflict

Two branches changed the same line of doc.txt. Merge them, detect the conflict, resolve it by keeping the 'Release' wording plus the author line, finish the merge commit, and report the merge parents, history and file content.

Starter code:

```
git init -q -b main demo && cd demo
git config user.name "Dev" && git config user.email "dev@example.com"
printf 'title: Draft\nauthor: Ada\n' > doc.txt
git add doc.txt && git commit -qm "base"

git switch -qc editor-a
sed -i.bak 's/Draft/Final/' doc.txt && rm doc.txt.bak
git commit -qam "retitle"

git switch -q main && git switch -qc editor-b
sed -i.bak 's/Draft/Release/' doc.txt && rm doc.txt.bak
git commit -qam "rename to release"

# merge editor-a into editor-b: both changed line 1
git merge editor-a >/dev/null 2>&1 || echo "conflict detected: $(git diff --name-only --diff-filter=U)"
# resolve: write doc.txt so it keeps the 'Release' wording and the author line, then stage and commit
# (finish the merge with the message "merge editor-a, keep Release")

echo "merge parents: $(git cat-file -p HEAD | grep -c '^parent')"
echo "history: $(git log --format=%s --no-merges | tr '\n' ',' | sed 's/,$//')"
cat doc.txt
```

Expected output:

```text
conflict detected: doc.txt
merge parents: 2
history: rename to release,retitle,base
title: Release
author: Ada
```

??? success "Show a solution"

    ```
    git init -q -b main demo && cd demo
    git config user.name "Dev" && git config user.email "dev@example.com"
    printf 'title: Draft\nauthor: Ada\n' > doc.txt
    git add doc.txt && git commit -qm "base"

    git switch -qc editor-a
    sed -i.bak 's/Draft/Final/' doc.txt && rm doc.txt.bak
    git commit -qam "retitle"

    git switch -q main && git switch -qc editor-b
    sed -i.bak 's/Draft/Release/' doc.txt && rm doc.txt.bak
    git commit -qam "rename to release"

    # merge editor-a into editor-b: both changed line 1
    git merge editor-a >/dev/null 2>&1 || echo "conflict detected: $(git diff --name-only --diff-filter=U)"
    # resolve: keep the 'Release' wording from this branch, add the author line unchanged
    printf 'title: Release\nauthor: Ada\n' > doc.txt
    git add doc.txt
    git commit -qm "merge editor-a, keep Release"

    echo "merge parents: $(git cat-file -p HEAD | grep -c '^parent')"
    echo "history: $(git log --format=%s --no-merges | tr '\n' ',' | sed 's/,$//')"
    cat doc.txt
    ```

    A conflict means both sides changed the same lines; you edit the file to the intended result, git add it and commit. The resulting merge commit has two parents and both branches' commits stay in the history. Verified by running the solution with real git.

### Cherry-pick, a bad reset and reflog recovery

Bring a hotfix onto main with cherry-pick, then simulate an accidental `git reset --hard HEAD~2` and recover the lost commits using the reflog.

Starter code:

```
git init -q -b main demo && cd demo
git config user.name "Dev" && git config user.email "dev@example.com"
for n in 1 2 3; do echo "line $n" >> notes.txt; git add notes.txt; git commit -qm "step $n"; done
git switch -qc hotfix
echo "fix" > fix.txt; git add fix.txt; git commit -qm "hotfix: patch"
git switch -q main

# 1) bring just the hotfix commit onto main without merging the whole branch (cherry-pick)
echo "after cherry-pick: $(git log --format=%s | tr '\n' ',' | sed 's/,$//')"

# 2) an accidental hard reset destroys the last two commits...
git reset -q --hard HEAD~2
echo "after reset:       $(git log --format=%s | tr '\n' ',' | sed 's/,$//')"

# 3) ...but the reflog still has them: restore main to where it was before the reset
echo "after recovery:    $(git log --format=%s | tr '\n' ',' | sed 's/,$//')"
```

Expected output:

```text
after cherry-pick: hotfix: patch,step 3,step 2,step 1
after reset:       step 2,step 1
after recovery:    hotfix: patch,step 3,step 2,step 1
```

??? success "Show a solution"

    ```
    git init -q -b main demo && cd demo
    git config user.name "Dev" && git config user.email "dev@example.com"
    for n in 1 2 3; do echo "line $n" >> notes.txt; git add notes.txt; git commit -qm "step $n"; done
    git switch -qc hotfix
    echo "fix" > fix.txt; git add fix.txt; git commit -qm "hotfix: patch"
    git switch -q main

    # 1) bring just the hotfix onto main without merging the whole branch
    git cherry-pick hotfix >/dev/null
    echo "after cherry-pick: $(git log --format=%s | tr '\n' ',' | sed 's/,$//')"

    # 2) an accidental hard reset destroys the last two commits...
    git reset -q --hard HEAD~2
    echo "after reset:       $(git log --format=%s | tr '\n' ',' | sed 's/,$//')"

    # 3) ...but the reflog still has them: restore main to where it was before the reset
    git reset -q --hard "HEAD@{1}"
    echo "after recovery:    $(git log --format=%s | tr '\n' ',' | sed 's/,$//')"
    ```

    Cherry-pick copies one commit's change onto the current branch; a hard reset only moves the branch pointer, so the reflog (HEAD@{1}) still lets you get back to the previous tip. Verified by running the solution with real git.

---

More free practice: the [GitHub & Git Mastery Path](index.md) curriculum, the [Interview Prep Pack](https://learnmasterypath.com/interview-pack/) and the [free resume ATS check](https://learnmasterypath.com/resume/).
