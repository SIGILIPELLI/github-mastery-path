# 02 · Rebase vs Merge

Both `git merge` and `git rebase` bring one branch's work into another, but
they produce different history shapes. Knowing when to use which — and
never mixing them the wrong way — is a core Git skill.

## Setting up divergent branches

```bash
git switch -c feature
echo f1 >> f.txt
git commit -aqm "feature commit 1"
echo f2 >> f.txt
git commit -aqm "feature commit 2"

git switch main
echo m1 >> g.txt
git add g.txt
git commit -qm "main commit 1"
```

At this point `main` and `feature` have diverged: `main` has one commit
`feature` doesn't, and `feature` has two commits `main` doesn't.

## Option A: merge (preserves history exactly as it happened)

```bash
git switch feature
git merge main
```

This creates a merge commit with two parents on `feature`, joining both
lines of development. The commits `feature commit 1` and `feature commit
2` keep their original hashes; nothing is rewritten. This is the safe
default for shared/public branches because it never changes history other
people may have already pulled.

## Option B: rebase (replays your commits on a new base)

Instead, from `feature`:

```bash
git rebase main
```
```
Rebasing (1/2)Rebasing (2/2)Successfully rebased and updated refs/heads/feature.
```

```bash
git log --oneline --graph --all
```
```
* 4284e21 feature commit 2
* 9ba91cc feature commit 1
* 1802e2e main commit 1
* 0eeac53 base
```

History is now perfectly linear — `feature`'s two commits sit right on top
of `main`'s latest commit, as if you'd branched off *after* that commit
existed. Notice: this rewrote `feature`'s commits. `feature commit 1` and
`feature commit 2` above have **new hashes** compared to before the
rebase, even though their content and messages are unchanged.

Merging `feature` back into `main` afterward is now a trivial
fast-forward — there's nothing to reconcile:

```bash
git switch main
git merge feature -m "Merge feature into main"
```
```
Updating 1802e2e..4284e21
Fast-forward (no commit created; -m option ignored)
 f.txt | 2 ++
 1 file changed, 2 insertions(+)
```

```bash
git log --oneline --graph --all
```
```
* 4284e21 feature commit 2
* 9ba91cc feature commit 1
* 1802e2e main commit 1
* 0eeac53 base
```

No merge commit at all — because after the rebase, `feature` already
contained everything `main` had plus more, so `main`'s pointer just moves.

## The rule that matters most: never rebase shared history

Because rebase **rewrites commit hashes**, rebasing a branch that other
people have already pulled and built on top of creates duplicate,
diverging history for them when they next pull — a genuinely painful mess
to untangle. The safe rule:

- Rebase freely on **your own local, not-yet-pushed** branches to clean up
  history before opening a PR.
- Once a branch is pushed and others may have it, prefer merging (or, if
  you must rebase a shared branch, coordinate with everyone and use
  `git push --force-with-lease`, covered in Level 3).

## Choosing between them in practice

| Situation | Prefer |
|---|---|
| Cleaning up your own messy WIP commits before a PR | rebase (interactive, Level 3) |
| Updating your feature branch with the latest `main` before opening/continuing a PR | rebase (if branch is still yours alone) or merge (if others share it) |
| Bringing a finished PR into `main` | merge (usually via GitHub's merge button) |
| Any branch already shared with teammates | merge |

## Rebase conflicts feel different from merge conflicts

A rebase conflict happens **per replayed commit**, not once for the whole
branch. If commit 2 of 5 conflicts, you resolve it, `git add` the files,
then `git rebase --continue` — and Git may stop again at commit 3. Abort
the whole thing at any point with `git rebase --abort`, which restores
`feature` to exactly where it was before you started.

## How It Actually Works

- **Merge** never touches existing commits. It computes a three-way diff
  from the merge base and writes **one new commit object** with two
  `parent` lines. Every commit that existed before still exists, unchanged,
  reachable from both branch tips.
- **Rebase** works commit by commit: for each commit on your branch (in
  order, oldest first), Git computes that commit's diff relative to *its
  own parent*, then re-applies that diff on top of the new base commit,
  and writes a **brand-new commit object** — new tree, new parent, and
  therefore a new SHA-1/SHA-256 hash, even though the author, message, and
  resulting file content are identical. The old commits aren't deleted
  immediately; they become unreferenced (no branch points at them) and
  are picked up by `git gc` later, but you can still find them for a
  while via `git reflog` if a rebase goes wrong.
- This is exactly why rebase is dangerous on shared branches: the commits
  your collaborators fetched still exist under their old hashes on their
  machines, while yours now has entirely different hash objects
  representing "the same" work — from Git's perspective these are
  unrelated commits that happen to have similar trees, not the same
  commits moved around.
- A fast-forward after rebase is possible because rebase's whole point is
  to make your branch's history look like it always descended linearly
  from the new base — so merging it back needs no three-way reconciliation
  at all, just a ref pointer update.

## Exercise

Recreate two diverging branches. Merge them one way and note the merge
commit's two parents with `git log --graph`. Reset that merge with
`git reset --hard` back to the pre-merge state, then instead rebase the
feature branch onto main and fast-forward merge it. Compare the two
resulting `git log --oneline --graph --all` outputs and describe, in your
own words, the structural difference.
