# 01 · What Is Git & Why Version Control?

**Version control** is a system that records changes to a set of files over
time so you can recall specific versions later, see who changed what and
when, and work on the same codebase as other people without overwriting each
other's work. Without it, "collaboration" usually means emailing zip files
named `project_final_v3_ACTUALLY_FINAL.zip` — and losing history the moment
someone forgets to rename a file correctly.

**Git** is the version control system almost everyone uses today. It was
created by Linus Torvalds in 2005 to manage the Linux kernel's source code —
a project with thousands of contributors — and has since become the
industry standard for every kind of software project, from solo scripts to
massive monorepos.

## Why Git specifically

There were version control systems before Git (CVS, Subversion/SVN,
Perforce). Git won out for a few concrete reasons:

- **Distributed, not centralized.** Every clone of a Git repository is a
  full copy of the project's entire history — not just the latest snapshot.
  You can commit, branch, and inspect history completely offline; you only
  need a network connection to sync with others.
- **Cheap, fast branching.** Creating a branch in Git is close to
  instantaneous and costs almost nothing in disk space, which makes
  branching a everyday habit rather than a rare, heavyweight event.
- **Data integrity.** Every piece of content, file, and commit is checksummed
  (using SHA-1, with migration to SHA-256 underway) before it's stored, and
  is referred to by that checksum. It's not possible to change a file or
  directory's contents without Git knowing about it.
- **Speed.** Most Git operations are local, so nearly everything — viewing
  history, diffing, branching, committing — is close to instant.

## What Git is *not*

Git is not GitHub. Git is the version control tool that runs on your
computer. **GitHub** (also GitLab, Bitbucket, etc.) is a website that hosts
Git repositories in the cloud and adds collaboration features on top — pull
requests, issue tracking, code review, CI/CD. You can use Git without ever
touching GitHub (many teams host their own Git servers), but in practice
almost everyone pairs the two. This course covers both: Git the tool
(Level 1–3 have plenty of pure-Git material) and GitHub the platform
(starting later in this level).

## The problem Git solves, concretely

Imagine three people editing the same `report.docx` by emailing it back and
forth. Two people make changes at the same time; whoever emails last "wins"
and the other person's work quietly vanishes. Now imagine the same three
people using Git:

- Each person has their own full copy of the project's history.
- Each person makes changes on their own schedule, and *commits* them —
  saving a labeled snapshot with a message explaining what and why.
- When they're ready, they *push* their commits to a shared location and
  *pull* everyone else's. Git tracks exactly which lines changed in which
  commit, so overlapping changes get merged automatically, and only genuine
  conflicts (the same line changed two different ways) need a human
  decision.
- At any point, anyone can look at the full history: who changed what
  line, when, and why (via the commit message).

## Key vocabulary you'll use constantly

| Term | Meaning |
|---|---|
| Repository ("repo") | A project's folder, plus all of its tracked history |
| Commit | A saved, labeled snapshot of the project at a point in time |
| Branch | An independent line of development, based on some commit |
| Working directory | The files on disk you're currently editing |
| Staging area (index) | Where you assemble what will go into the *next* commit |
| Remote | A copy of the repository hosted elsewhere (e.g. on GitHub) |
| Clone | A full local copy of a remote repository, history included |

You'll meet all of these hands-on in the next few modules — this one is
purely conceptual so the rest of the level has solid footing.

## How It Actually Works

The "cheap, fast branching" and "data integrity" claims above aren't
marketing — they fall directly out of one design decision: Git doesn't
store *diffs* between versions of files. It stores **snapshots**, addressed
by content.

- Every version of every file (a **blob**), every directory listing (a
  **tree**), and every commit is run through SHA-1, producing a 40-character
  hex digest. That digest *is* the object's name and its address in
  storage — `.git/objects/<first 2 chars>/<remaining 38 chars>`.
- Because the hash is derived purely from content, two files with identical
  bytes anywhere in the repo — even in different commits, different
  branches, different directories — are stored *once*. This is why creating
  a branch costs almost nothing: a branch is just a 41-byte text file
  (`.git/refs/heads/<name>`) holding a commit's SHA. No files are copied.
- A commit object doesn't store a diff either — it points to one tree (the
  full snapshot of the project at that moment) plus the SHA of its parent
  commit(s), an author, a committer, and a message. Walking `parent` pointers
  backward from any commit is what "history" *is* — there's no separate
  history database, just a chain of hash pointers, which is why Git calls
  this structure a **DAG** (directed acyclic graph) of commits.
- Content-addressing is also the integrity guarantee: if a single bit in a
  blob changes, its SHA changes, which changes the tree that references it,
  which changes every commit downstream. You cannot silently corrupt or
  rewrite history without the hashes revealing it — this is the same
  mechanism (a Merkle tree) used by systems like Bitcoin and IPFS.

You'll see these objects directly with `git cat-file` and `git hash-object`
in Level 3's "Git Internals" module — for now, just know that everything
Git does fast (branch, checkout, diff, log) is fast *because* it's built on
hash-addressed snapshots rather than a linear list of patches.

## Exercise

Without touching a keyboard yet: write down, in your own words, one
sentence each for what a *commit*, a *branch*, and a *remote* are. Then
write one sentence explaining the difference between Git and GitHub. Keep
these three sentences somewhere — you'll be able to check them against your
own understanding again after finishing Level 1.
