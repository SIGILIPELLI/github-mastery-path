---
title: "Final Quiz"
description: "Test what you have learned across all four levels of GitHub \u0026 Git Mastery Path with a 18-question final quiz."
---

# Final Quiz

18 questions drawn from all four levels of this course. You need **70% (13 out of 18)** to pass —
you can retake it as many times as you like. Your best result is saved in this browser.

<div id="mp-quiz-root"></div>

<script type="application/json" id="mp-quiz-data">
[
  {
    "question": "What distinguishes Git from GitHub?",
    "options": [
      "GitHub is the tool",
      "Git is the distributed version control tool; GitHub hosts repositories and adds collaboration features",
      "Git is hosted",
      "They're the same"
    ],
    "correctIndex": 1,
    "explanation": "Git works fully offline."
  },
  {
    "question": "What does git status --short show? (in a new empty repository, identity configured, default branch main)\n```bash\necho hi > a.txt\ngit add a.txt\necho more > b.txt\ngit status --short\n```\n",
    "options": [
      "A  a.txt\nA  b.txt",
      "M  a.txt",
      "?? a.txt\n?? b.txt",
      "A  a.txt\n?? b.txt"
    ],
    "correctIndex": 3,
    "explanation": "A = staged new file; ?? = untracked. Verified by running it with real git."
  },
  {
    "question": "After this, which commit does main point to and what does feature have? (in a new empty repository, identity configured, default branch main)\n```bash\necho a>f; git add f; git commit -qm c1\ngit switch -qc feature\necho b>>f; git commit -qam c2\ngit switch -q main\ngit log --format=%s | paste -sd' ' -; git log feature --format=%s | paste -sd' ' -\n```\n",
    "options": [
      "c2\nc2 c1",
      "c1\nc2 c1",
      "c1\nc1",
      "c2 c1\nc2 c1"
    ],
    "correctIndex": 1,
    "explanation": "A branch is just a pointer; main still points at c1. Verified by running it with real git."
  },
  {
    "question": "How many parents does the commit made by `git merge --no-ff` have? (in a new empty repository, identity configured, default branch main)\n```bash\necho a>f; git add f; git commit -qm c1\ngit switch -qc feat\necho b>>f; git commit -qam c2\ngit switch -q main\ngit merge -q --no-ff feat -m 'merge feat'\ngit cat-file -p HEAD | grep -c '^parent'\n```\n",
    "options": [
      "0",
      "3",
      "2",
      "1"
    ],
    "correctIndex": 2,
    "explanation": "Even when fast-forward is possible, --no-ff creates a merge commit with two parents. Verified by running it with real git."
  },
  {
    "question": "What does the conflicted file contain? (in a new empty repository, identity configured, default branch main)\n```bash\nprintf 'base\\n'>f; git add f; git commit -qm base\ngit switch -qc a; echo A>f; git commit -qam a\ngit switch -q main; git switch -qc b; echo B>f; git commit -qam b\ngit merge a >/dev/null 2>&1 || true\ncat f\n```\n",
    "options": [
      "B\nA",
      "A\nB",
      "<<<<<<< HEAD\nB\n=======\nA\n>>>>>>> a",
      "base"
    ],
    "correctIndex": 2,
    "explanation": "The current branch (HEAD) comes first, the incoming branch last. Verified by running it with real git."
  },
  {
    "question": "What does the history look like after rebasing feat onto main? (in a new empty repository, identity configured, default branch main)\n```bash\necho 1 > f; git add f; git commit -qm base\ngit switch -qc feat; echo f>g; git add g; git commit -qm feat\ngit switch -q main; echo m>h; git add h; git commit -qm main2\ngit switch -q feat; git rebase -q main\ngit log --format=%s | tr '\\n' ' '\n```\n",
    "options": [
      "feat main2 base",
      "base",
      "feat base main2",
      "main2 feat base"
    ],
    "correctIndex": 0,
    "explanation": "Rebase replays feat's commits on top of main, producing linear history. Verified by running it with real git."
  },
  {
    "question": "Which commit does cherry-pick copy? (in a new empty repository, identity configured, default branch main)\n```bash\necho a>f; git add f; git commit -qm base\ngit switch -qc other; echo b>g; git add g; git commit -qm 'add g'\ngit switch -q main\ngit cherry-pick other >/dev/null\ngit log --format=%s | tr '\\n' ' '\n```\n",
    "options": [
      "add g base",
      "base",
      "add g",
      "other base"
    ],
    "correctIndex": 0,
    "explanation": "Cherry-pick applies that commit's change as a new commit on the current branch. Verified by running it with real git."
  },
  {
    "question": "How do you get a commit back after `git reset --hard HEAD~1`? (in a new empty repository, identity configured, default branch main)\n```bash\necho a>f; git add f; git commit -qm one\necho b>>f; git commit -qam two\ngit reset -q --hard HEAD~1\ngit reset -q --hard HEAD@{1}\ngit log --format=%s | head -1\n```\n",
    "options": [
      "Lost forever",
      "one",
      "Error",
      "two"
    ],
    "correctIndex": 3,
    "explanation": "The reflog records where HEAD pointed, so the commit is still reachable. Verified by running it with real git."
  },
  {
    "question": "What happens to a commit when a pre-commit hook exits non-zero? (in a new empty repository, identity configured, default branch main)\n```bash\necho a>f; git add f\nprintf '#!/bin/sh\\necho blocked; exit 1\\n' > .git/hooks/pre-commit; chmod +x .git/hooks/pre-commit\ngit commit -qm c 2>&1 | head -1; git rev-list --all --count\n```\n",
    "options": [
      "Error",
      "blocked\n0",
      "(none)\n1",
      "blocked\n1"
    ],
    "correctIndex": 1,
    "explanation": "A failing hook aborts the commit. Verified by running it with real git."
  },
  {
    "question": "Which commit does bisect report as the first bad one? (in a new empty repository, identity configured, default branch main)\n```bash\nfor i in 1 2 3 4 5 6; do echo $i > v; if [ $i -ge 4 ]; then echo bug > b; fi; git add -A; git commit -qm \"c$i\"; done\ngit bisect start >/dev/null\ngit bisect bad HEAD >/dev/null\ngit bisect good HEAD~5 >/dev/null\ngit bisect run sh -c '! test -f b' >/dev/null\ngit log --format=%s -1 refs/bisect/bad\ngit bisect reset >/dev/null 2>&1\n```\n",
    "options": [
      "c3",
      "c4",
      "c1",
      "c5"
    ],
    "correctIndex": 1,
    "explanation": "Binary search over history, O(log n). Verified by running it with real git."
  },
  {
    "question": "`git push` is rejected because the remote has new commits. What do you do?",
    "options": [
      "Reclone",
      "Delete remote",
      "Force push",
      "Fetch and integrate (merge or rebase) the remote changes, then push"
    ],
    "correctIndex": 3,
    "explanation": "Never force-push shared branches casually."
  },
  {
    "question": "You rebased a branch you already pushed. What push do you need?",
    "options": [
      "Normal push",
      "A force push, preferably --force-with-lease, and only on your own branch",
      "Pull",
      "Merge"
    ],
    "correctIndex": 1,
    "explanation": "History was rewritten."
  },
  {
    "question": "A committed .env file leaked a key. What must you do besides deleting the file?",
    "options": [
      "Nothing",
      "Revoke/rotate the key; Git history still contains it",
      "Force-push only",
      "Rename"
    ],
    "correctIndex": 1,
    "explanation": "Treat it as compromised."
  },
  {
    "question": "Main must never receive unreviewed or failing code. What do you configure?",
    "options": [
      "A README",
      "Labels",
      "Branch protection: required reviews, required status checks and no direct pushes",
      "Tags"
    ],
    "correctIndex": 2,
    "explanation": "Server-side enforcement."
  },
  {
    "question": "A workflow deploys from forks' PRs with secrets and an attacker steals them. What went wrong?",
    "options": [
      "Cache",
      "Secrets were exposed to untrusted code (pull_request_target/misconfig); restrict secrets and permissions",
      "Tags",
      "Bad runner"
    ],
    "correctIndex": 1,
    "explanation": "Least privilege for tokens."
  },
  {
    "question": "Why pin third-party actions to a commit SHA?",
    "options": [
      "Style",
      "Tags can be moved to malicious code; a SHA is immutable",
      "Law",
      "Speed"
    ],
    "correctIndex": 1,
    "explanation": "Supply-chain safety."
  },
  {
    "question": "A monorepo's CI runs everything for every change. What helps?",
    "options": [
      "Bigger runner",
      "Path filters/affected-project detection and caching",
      "Skip tests",
      "Run all"
    ],
    "correctIndex": 1,
    "explanation": "Scale CI."
  },
  {
    "question": "An old release needs a security fix while main has moved on. What do you do?",
    "options": [
      "Release main",
      "Force",
      "Skip",
      "Fix on a release branch (or cherry-pick the fix), tag a patch release"
    ],
    "correctIndex": 3,
    "explanation": "Backport."
  }
]
</script>

## Coding challenges

Try these on your own computer, then compare your output. Nothing you write is sent anywhere.

### Resolve a merge conflict

Two branches changed the same line of doc.txt. Merge them, detect the conflict, resolve it by keeping the 'Release' wording plus the author line, finish the merge commit, and report the merge parents, history and file content.

**Starter code**

```yaml
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

**Expected output**

```text
conflict detected: doc.txt
merge parents: 2
history: rename to release,retitle,base
title: Release
author: Ada
```

<details><summary>Show solution and explanation</summary>

<pre><code>git init -q -b main demo &amp;&amp; cd demo
git config user.name &quot;Dev&quot; &amp;&amp; git config user.email &quot;dev@example.com&quot;
printf &#x27;title: Draft\nauthor: Ada\n&#x27; &gt; doc.txt
git add doc.txt &amp;&amp; git commit -qm &quot;base&quot;

git switch -qc editor-a
sed -i.bak &#x27;s/Draft/Final/&#x27; doc.txt &amp;&amp; rm doc.txt.bak
git commit -qam &quot;retitle&quot;

git switch -q main &amp;&amp; git switch -qc editor-b
sed -i.bak &#x27;s/Draft/Release/&#x27; doc.txt &amp;&amp; rm doc.txt.bak
git commit -qam &quot;rename to release&quot;

# merge editor-a into editor-b: both changed line 1
git merge editor-a &gt;/dev/null 2&gt;&amp;1 || echo &quot;conflict detected: $(git diff --name-only --diff-filter=U)&quot;
# resolve: keep the &#x27;Release&#x27; wording from this branch, add the author line unchanged
printf &#x27;title: Release\nauthor: Ada\n&#x27; &gt; doc.txt
git add doc.txt
git commit -qm &quot;merge editor-a, keep Release&quot;

echo &quot;merge parents: $(git cat-file -p HEAD | grep -c &#x27;^parent&#x27;)&quot;
echo &quot;history: $(git log --format=%s --no-merges | tr &#x27;\n&#x27; &#x27;,&#x27; | sed &#x27;s/,$//&#x27;)&quot;
cat doc.txt
</code></pre>

<p>A conflict means both sides changed the same lines; you edit the file to the intended result, git add it and commit. The resulting merge commit has two parents and both branches&#x27; commits stay in the history. Verified by running the solution with real git.</p>
</details>

### Cherry-pick, a bad reset and reflog recovery

Bring a hotfix onto main with cherry-pick, then simulate an accidental `git reset --hard HEAD~2` and recover the lost commits using the reflog.

**Starter code**

```yaml
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

**Expected output**

```text
after cherry-pick: hotfix: patch,step 3,step 2,step 1
after reset:       step 2,step 1
after recovery:    hotfix: patch,step 3,step 2,step 1
```

<details><summary>Show solution and explanation</summary>

<pre><code>git init -q -b main demo &amp;&amp; cd demo
git config user.name &quot;Dev&quot; &amp;&amp; git config user.email &quot;dev@example.com&quot;
for n in 1 2 3; do echo &quot;line $n&quot; &gt;&gt; notes.txt; git add notes.txt; git commit -qm &quot;step $n&quot;; done
git switch -qc hotfix
echo &quot;fix&quot; &gt; fix.txt; git add fix.txt; git commit -qm &quot;hotfix: patch&quot;
git switch -q main

# 1) bring just the hotfix onto main without merging the whole branch
git cherry-pick hotfix &gt;/dev/null
echo &quot;after cherry-pick: $(git log --format=%s | tr &#x27;\n&#x27; &#x27;,&#x27; | sed &#x27;s/,$//&#x27;)&quot;

# 2) an accidental hard reset destroys the last two commits...
git reset -q --hard HEAD~2
echo &quot;after reset:       $(git log --format=%s | tr &#x27;\n&#x27; &#x27;,&#x27; | sed &#x27;s/,$//&#x27;)&quot;

# 3) ...but the reflog still has them: restore main to where it was before the reset
git reset -q --hard &quot;HEAD@{1}&quot;
echo &quot;after recovery:    $(git log --format=%s | tr &#x27;\n&#x27; &#x27;,&#x27; | sed &#x27;s/,$//&#x27;)&quot;
</code></pre>

<p>Cherry-pick copies one commit&#x27;s change onto the current branch; a hard reset only moves the branch pointer, so the reflog (HEAD@{1}) still lets you get back to the previous tip. Verified by running the solution with real git.</p>
</details>
