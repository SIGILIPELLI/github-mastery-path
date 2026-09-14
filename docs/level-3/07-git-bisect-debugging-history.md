# 07 · Git Bisect & Debugging History

`git bisect` finds the exact commit that introduced a bug using binary
search over your commit history — instead of manually checking out
commits one by one, Git narrows a range of N commits down to the culprit
in roughly log₂(N) steps.

## Setting up a real regression

A tiny calculator script, correct for the first few commits, broken
starting at commit 4:

```bash
cat > calc.sh <<'EOF'
#!/bin/sh
echo $(( $1 + $2 ))
EOF
chmod +x calc.sh
git add calc.sh && git commit -m "commit 1: working add"
# ... two harmless commits ...
sed -i '' 's/+/\*/' calc.sh
git commit -am "commit 4: BUG introduced (uses * instead of +)"
# ... two more harmless commits ...
git log --oneline
```
```
cdb6187 commit 6: noop change
c13e720 commit 5: noop change
1d9dd9a commit 4: BUG introduced (uses * instead of +)
4a5eed5 commit 3: noop change
ce69b05 commit 2: noop change
ce66c86 commit 1: working add
```

Pretend you only know: "it works at commit 1, it's broken now at HEAD" —
you don't know which of commits 2-6 broke it.

## Manual bisect walkthrough

```bash
git bisect start
git bisect bad HEAD
git bisect good ce66c86
```
```
status: waiting for good commit(s), bad commit known
Bisecting: 2 revisions left to test after this (roughly 1 step)
[4a5eed5...] commit 3: noop change
```

Git checked out the midpoint commit automatically. You test it — here,
`./calc.sh 2 3` correctly prints `5` — and tell Git:

```bash
git bisect good
```

Git checks out the next midpoint. Repeat: test, then `git bisect good` or
`git bisect bad`, until Git announces the answer.

## Automating it with a test script

Since "does the bug reproduce" can often be scripted, `git bisect run`
does the entire loop for you:

```bash
cat > /tmp/test_calc.sh <<'EOF'
#!/bin/sh
result=$(./calc.sh 2 3)
[ "$result" = "5" ]
EOF
chmod +x /tmp/test_calc.sh

git bisect start
git bisect bad HEAD
git bisect good ce66c86
git bisect run /tmp/test_calc.sh
```
```
Bisecting: 2 revisions left to test after this (roughly 1 step)
[4a5eed5b1dfa6246938195791409473b885ca34c] commit 3: noop change
running '/tmp/test_calc.sh'
Bisecting: 0 revisions left to test after this (roughly 1 step)
[c13e720abb8e2ef84affadd1314af076f3e6bc0c] commit 5: noop change
running '/tmp/test_calc.sh'
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[1d9dd9a9d219e77d5e35b70f031682b8817393ea] commit 4: BUG introduced (uses * instead of +)
running '/tmp/test_calc.sh'
1d9dd9a9d219e77d5e35b70f031682b8817393ea is the first bad commit
commit 1d9dd9a9d219e77d5e35b70f031682b8817393ea
Author: You <you@example.com>

    commit 4: BUG introduced (uses * instead of +)

 calc.sh | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

Six commits, three actual checkouts, exact culprit identified with zero
manual guessing.

## Reviewing and replaying the whole search

```bash
git bisect log
```
```
git bisect start
git bisect bad cdb6187...
git bisect good ce66c86...
git bisect good 4a5eed5...
git bisect bad c13e720...
git bisect bad 1d9dd9a...
# first bad commit: [1d9dd9a...] commit 4: BUG introduced (uses * instead of +)
```

This log is itself a replayable script (`git bisect replay <file>`) —
useful for sharing exactly how a regression was tracked down in a bug
report or postmortem.

## Cleaning up

```bash
git bisect reset
```
```
Previous HEAD position was 1d9dd9a commit 4: BUG introduced...
Switched to branch 'main'
```

Always run this when done — bisect leaves you in a detached HEAD on
whatever commit was last checked out, and `reset` returns you to the
branch you started from.

## How It Actually Works

- `git bisect` operates purely on the **commit graph's ancestry
  ordering**, not dates or commit messages — it computes the midpoint
  between the known-good and known-bad commits by counting commits along
  the reachable path between them and checking out the one halfway, using
  the same graph-walking logic `git log`/`git rev-list` use internally.
- Each `good`/`bad` verdict shrinks the search range by roughly half,
  which is exactly why the number of checkouts needed grows only
  logarithmically with history size — bisecting a 10,000-commit range
  takes about 14 steps, not 10,000.
- `git bisect run <script>` treats the script's **exit code** as the
  verdict, identical to how hooks work: exit 0 means "good," any nonzero
  exit code 1-127 (excluding 125, reserved to mean "skip this commit, it
  can't be tested") means "bad." This is why the test script above is a
  plain shell conditional — its own exit status *is* the bisect answer.
- Internally, bisect state lives in `.git/BISECT_LOG`,
  `.git/BISECT_START`, and refs like `refs/bisect/bad` — ordinary Git
  refs and files, nothing exotic — which is exactly what `git bisect
  reset` cleans up, and what makes `git bisect log`'s replayable output
  possible: it's just reconstructing the sequence of ref updates that
  happened.

## Exercise

Build a repo with 8 commits where a script's behavior breaks at commit 5,
with no other clue than "works at commit 1, broken at HEAD." Write a
one-line test script whose exit code reflects correct/incorrect behavior,
and use `git bisect start` + `git bisect run` to find the exact breaking
commit automatically. Confirm the same answer with `git bisect log`.
