# 06 · Remotes (clone/push/pull/fetch)

Everything so far has happened on one machine. A **remote** is a version of
your repository hosted somewhere else — most commonly on GitHub — that you
sync with. This module covers the mechanics using a plain local "remote"
first (no GitHub account needed yet); the next few modules connect this to
an actual GitHub repository.

## Cloning: getting a full copy

`git clone` downloads a repository's entire history into a new folder and
automatically sets up a remote called `origin` pointing back at the source:

```bash
git clone https://github.com/octocat/Hello-World.git
cd Hello-World
git remote -v
```
```
origin  https://github.com/octocat/Hello-World.git (fetch)
origin  https://github.com/octocat/Hello-World.git (push)
```

A clone is a real, independent local repository — every commit, branch, and
tag comes with it. You could delete the original entirely and this clone
would still have the full history.

## Adding a remote to an existing repo

If you started a project locally (`git init`) and want to connect it to a
remote afterward:

```bash
git remote add origin https://github.com/yourname/your-repo.git
git remote -v
```
```
origin  https://github.com/yourname/your-repo.git (fetch)
origin  https://github.com/yourname/your-repo.git (push)
```

`origin` is just a conventional name for "the primary remote" — you can
name it anything, and you can have multiple remotes (e.g. `origin` for your
fork, `upstream` for the original project — covered in module 10).

## `git push` — send your commits to the remote

```bash
git push -u origin main
```
```
Enumerating objects: 6, done.
...
To https://github.com/yourname/your-repo.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

`-u` (`--set-upstream`) links your local `main` to `origin/main` so future
pushes/pulls on this branch can drop the arguments:

```bash
git push          # after the first -u push, this is enough
```

## `git fetch` — download without merging

```bash
git fetch origin
```
```
From https://github.com/yourname/your-repo
   48f9b4e..986424b  main       -> origin/main
```

`fetch` downloads new commits from the remote into a local *remote-tracking
branch* called `origin/main` — but it does **not** touch your own `main`
branch or your working directory. This is the safe way to see what's new
without changing anything yet:

```bash
git log --oneline main..origin/main   # commits on origin/main you don't have locally
git diff main origin/main             # what actually changed
```

## `git pull` — fetch, then merge

```bash
git pull origin main
```

`git pull` is shorthand for `git fetch` followed by `git merge
origin/main` into your current branch — it brings the remote's changes into
your working directory in one step. If your local branch has diverged from
the remote (you both have new commits), this triggers the same three-way
merge (or conflict) behavior from the previous module.

```bash
git pull
```
```
From https://github.com/yourname/your-repo
   48f9b4e..986424b  main       -> origin/main
Updating 48f9b4e..986424b
Fast-forward
 notes.txt | 1 +
 1 file changed, 1 insertion(+)
```

A common team preference is `git pull --rebase`, which replays your local
commits on top of the fetched ones instead of merging — producing a linear
history. Rebase is covered in depth in Level 2; for now, plain `git pull`
is perfectly fine.

## What "push rejected" means

```bash
git push
```
```
! [rejected]        main -> main (fetch first)
error: failed to push some refs to '...'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally.
```

This means someone else pushed commits to the remote that you don't have
yet. The fix is always the same: `git pull` first (resolving any conflicts
that come up), then `git push` again. Git refuses to silently overwrite
remote history — this rejection is a safety feature, not a bug.

## Remote-tracking branches, visualized

```
Local repo                          Remote (origin)
-----------                         ----------------
main         <-- your work           main
origin/main  <-- last-known state    (moves as others push)
             of origin's main
```

`origin/main` only updates when you `fetch` or `pull` — it's a snapshot of
what the remote looked like the last time you checked in, not a live view.

## Worked example: full round trip

```bash
# Machine A
git init -b main
git remote add origin /path/to/shared/repo.git
echo "v1" > file.txt && git add -A && git commit -m "v1"
git push -u origin main

# Machine B
git clone /path/to/shared/repo.git
cd repo
echo "v2" >> file.txt && git add -A && git commit -m "v2"
git push

# Machine A, catching up
git pull
cat file.txt
# v1
# v2
```

## How It Actually Works

`fetch`, `push`, and `pull` are just object-database synchronization over a
network protocol — nothing about a remote repository is magic once you see
what's actually being transferred:

- A remote is a named URL stored in `.git/config` under
  `[remote "origin"]`. `git remote add origin <url>` only writes that
  config entry — it makes no network call.
- `git fetch` opens a connection and runs Git's **smart HTTP/SSH
  protocol**: your client says "here's what commit SHAs I already have,"
  the server replies with the SHAs of its branch tips, and your client
  computes exactly which objects (commits, trees, blobs) it's missing —
  then the server packs *only those* objects into a single **packfile**
  (a compressed, delta-encoded bundle) and streams it over. Fetch then
  writes the received objects into your local `.git/objects/` and updates
  your **remote-tracking branches** — `refs/remotes/origin/main` — to match
  what it just saw. Your own `refs/heads/main` is deliberately *not*
  touched by fetch; that's why `origin/main` can lag behind or ahead of
  `main` until you merge/pull.
- `git push` runs the same negotiation in reverse: your client tells the
  server "I want your `refs/heads/main` to become SHA X," sends a packfile
  of any objects the server doesn't already have, and the server only
  accepts the ref update if it's a **fast-forward** of what it currently
  has — i.e. the SHA you're pushing has the server's current tip as an
  ancestor. If someone else pushed in the meantime, the server's tip moved,
  your update is no longer a fast-forward, and the push is rejected — this
  is the exact mechanism behind the "non-fast-forward" error, and it's
  what forces you to `pull` (fetch + merge/rebase) before you can push
  again.
- `git pull` is literally shorthand for `git fetch` followed by
  `git merge origin/<branch>` (or `git rebase` with `--rebase`) — there is
  no separate "pull" logic in Git's object model, just those two familiar
  operations run back to back.

## Exercise

Create two local folders to simulate two "machines": `git init --bare
shared.git` in one location to act as the remote, then clone it twice into
`machine-a` and `machine-b`. From `machine-a`, commit a file and push. From
`machine-b`, pull, add a second file, and push. Finally, go back to
`machine-a` and pull again — confirm you now see both files.
