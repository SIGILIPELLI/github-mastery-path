# 01 · Merge Conflicts & Resolution

Level 1 introduced conflicts briefly. Here's the full workflow: how a
conflict happens, how to read the markers, how to resolve with tools, and
how to bail out safely.

## Reproducing a real conflict

Starting from a shared `notes.txt` on `main`, create a branch and edit the
same line on both sides:

```bash
git switch -c feature/x
echo "feature line" >> notes.txt
git commit -aqm "Feature edit line 1"

git switch main
sed -i '' '1s/.*/line one (main edit)/' notes.txt
git commit -aqm "Main edit line 1"

git merge feature/x
```
```
Auto-merging notes.txt
CONFLICT (content): Merge conflict in notes.txt
Automatic merge failed; fix conflicts and then commit the result.
```

```bash
git status
```
```
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   notes.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

`notes.txt` now contains both versions with conflict markers:

```
<<<<<<< HEAD
line one (main edit)
=======
line one
feature line
>>>>>>> feature/x
```

Reading this: everything between `<<<<<<< HEAD` and `=======` is **your
current branch's version**; everything between `=======` and
`>>>>>>> feature/x` is the **incoming branch's version**. Git couldn't
decide which line 1 was "correct" because both sides touched it.

## Resolving it

Edit the file to the content you actually want and remove every marker
line — Git does not remove them for you:

```bash
printf 'line one (main edit)\nfeature line\n' > notes.txt
cat notes.txt
```
```
line one (main edit)
feature line
```

Then stage and finish the merge commit:

```bash
git add notes.txt
git commit -m "Merge feature/x, resolve notes.txt conflict"
```

```bash
git log --oneline --graph --all
```
```
*   b7effe1 Merge feature/x, resolve notes.txt conflict
|\
| * 42200fa Feature edit line 1
* | e3bb258 Main edit line 1
|/
* 7a7d91c Add notes.txt
```

Staging a previously-conflicted file is exactly how Git knows resolution
is complete for that file — `git commit` refuses to finish a merge while
any path still shows as unmerged in `git status`.

## Choosing one side wholesale

If you know the whole file should just be "theirs" or "ours" (not a
line-by-line merge), skip manual editing:

```bash
git checkout --ours notes.txt    # keep current branch's version entirely
git checkout --theirs notes.txt  # keep incoming branch's version entirely
git add notes.txt
```

Use this carefully — it discards the other side's changes to that file
completely, not just the conflicting lines.

## Conflict markers on more than two lines

Real conflicts often span whole blocks, not single lines, and a file can
have several conflict regions at once. Search for them systematically
rather than trusting your editor's diff view alone:

```bash
grep -n "^<<<<<<<\|^=======\|^>>>>>>>" notes.txt
```

Don't consider a file resolved until this returns nothing.

## Diff3 style: seeing the common ancestor too

By default conflict markers show only "ours" vs "theirs." Turning on the
`diff3` conflict style adds the original (merge-base) content in the
middle, which often makes it obvious what each side actually *changed*:

```bash
git config merge.conflictstyle diff3
```

A conflict then looks like:

```
<<<<<<< HEAD
line one (main edit)
||||||| 7a7d91c
line one
=======
line one
feature line
>>>>>>> feature/x
```

Now you can see the original line was `line one`, main changed it in
place, and feature/x left it alone but added a second line — so the
correct resolution (keep the edit, keep the addition) is clear.

## Aborting instead of resolving

If you started a merge and want to back out entirely:

```bash
git merge --abort
```

This is safe any time before you run `git commit` to finish the merge — it
resets the index and working tree to match `HEAD` exactly and clears
`.git/MERGE_HEAD`.

## Conflicts during rebase vs merge

The markers look identical, but the recovery command differs:

| Situation      | Abort command      | Continue command    |
|-----------------|--------------------|----------------------|
| `git merge`     | `git merge --abort`| `git commit`         |
| `git rebase`    | `git rebase --abort`| `git rebase --continue` |
| `git cherry-pick` | `git cherry-pick --abort` | `git cherry-pick --continue` |

Using the wrong pair (e.g. `git commit` mid-rebase) leaves things in a
confusing state, so always check `git status` — it prints the exact next
command to run at the bottom of its output.

## How It Actually Works

A merge conflict is what happens when Git's three-way diff can't pick a
winner automatically, and the conflict markers are literally what gets
written into the working-tree file and the index.

- Git computes the merge base (the common ancestor), then diffs
  `merge-base → ours` and `merge-base → theirs` for every file, line by
  line (technically hunk by hunk, using the same diff algorithm as
  `git diff`).
- If only one side changed a given hunk, Git applies that side's change
  automatically — no conflict, no markers, nothing to review.
- If **both** sides changed the *same* hunk (overlapping line ranges),
  Git can't merge them textually. It writes a synthetic combined version
  into the file — `<<<<<<< HEAD` / your content / `=======` / their
  content / `>>>>>>>` — and leaves the file **unmerged** in the index.
- "Unmerged" is a real index state, not just a working-tree annotation:
  each conflicted path gets three index entries simultaneously (stage 1 =
  common ancestor, stage 2 = ours, stage 3 = theirs), which is why
  `git checkout --ours <file>` and `--theirs <file>` work — they just pull
  stage 2 or stage 3 back into the working tree and collapse it to a
  single normal index entry.
- Running `git add <file>` on a conflicted path removes stages 2/3 and
  writes a single new blob for the resolved content at stage 0 (normal).
  `git commit` checks that no path still has stage-1/2/3 entries before it
  will let the merge commit be created — that's the actual mechanism
  behind "you have unmerged paths."
- `diff3` style works because Git already has the merge-base blob in the
  object database (it needed it to compute the diff in the first place);
  turning the setting on just tells Git to also print that blob's content
  between `||||||| <base>` and `=======` instead of discarding it.

## Exercise

Recreate the scenario above in a scratch repo: two branches editing the
same line of the same file. Trigger the conflict, enable
`git config merge.conflictstyle diff3`, redo the merge, and use the extra
ancestor context to resolve it. Confirm with `git log --graph --oneline
--all` that the result is a single merge commit with two parents.
