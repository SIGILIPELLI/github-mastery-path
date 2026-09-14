# 06 · Git Hooks

Hooks are scripts Git runs automatically at specific points in its
workflow (before a commit, before a push, after a checkout). They live in
every repo's `.git/hooks/` directory and are entirely local — nothing
about them is transmitted by `clone`, `push`, or `pull`.

## Where hooks live

```bash
ls .git/hooks/
```
```
applypatch-msg.sample
commit-msg.sample
pre-commit.sample
pre-push.sample
... (all installed with a .sample extension, disabled by default)
```

Git ships every hook as a `.sample` file with example shell code — rename
(remove `.sample`) and make it executable to activate it.

## A real pre-commit hook that blocks bad commits

```bash
cat > .git/hooks/pre-commit <<'EOF'
#!/bin/sh
if git diff --cached | grep -qi "TODO"; then
  echo "Commit rejected: found a TODO in staged changes."
  exit 1
fi
EOF
chmod +x .git/hooks/pre-commit
```

```bash
echo "code with TODO: fix later" > f.txt
git add f.txt
git commit -m "add file with todo"
```
```
Commit rejected: found a TODO in staged changes.
```

The commit never happened — `git log` shows nothing new. Fixing the
content and retrying:

```bash
echo "clean code" > f.txt
git add f.txt
git commit -m "add clean file"
```
```
[main (root-commit) 944c3bd] add clean file
 1 file changed, 1 insertion(+)
 create mode 100644 f.txt
```

This time the hook's `grep` found nothing, exited 0, and Git proceeded
normally.

## The hooks that matter most

| Hook | Runs | Typical use |
|---|---|---|
| `pre-commit` | before a commit is created, after staging | linting, formatting checks, blocking secrets |
| `commit-msg` | after the message is written, before commit finalizes | enforcing a message format (e.g. Conventional Commits) |
| `pre-push` | before `git push` sends anything | running the full test suite, blocking force-pushes to protected local branches |
| `post-checkout` | after `git checkout`/`switch` | reinstalling dependencies if `package-lock.json` changed |
| `post-merge` | after a successful merge/pull | same, for merges |

A `commit-msg` hook receives the message file path as its one argument
and can reject the commit by exiting non-zero:

```bash
cat > .git/hooks/commit-msg <<'EOF'
#!/bin/sh
if ! grep -qE '^(feat|fix|docs|chore|refactor|test)(\(.+\))?: ' "$1"; then
  echo "Commit message must start with feat:, fix:, docs:, etc."
  exit 1
fi
EOF
chmod +x .git/hooks/commit-msg
```

## The problem with local-only hooks — and the fix

Hooks in `.git/hooks/` are **never committed or cloned** — every
teammate would have to manually install the same script, which doesn't
scale. Two real solutions:

1. **Husky** (Node ecosystem) or similar tools store hook scripts in a
   tracked directory (e.g. `.husky/`) and wire them up via an npm
   `postinstall` step, so `npm install` sets up hooks for everyone
   automatically.
2. **`core.hooksPath`** — point Git at a tracked directory instead of
   `.git/hooks/` directly:

   ```bash
   git config core.hooksPath .githooks
   mkdir .githooks
   cp .git/hooks/pre-commit .githooks/pre-commit
   git add .githooks/pre-commit
   git commit -m "Add shared pre-commit hook"
   ```

   Each teammate still has to run the `git config` line once (it's a
   local setting), but the hook's actual content is now versioned and
   reviewed like any other file.

## Bypassing a hook when you genuinely need to

```bash
git commit --no-verify -m "emergency fix, bypassing lint hook"
```

Use sparingly and deliberately — a hook exists to catch something, and
routinely bypassing it defeats the purpose.

## How It Actually Works

- A hook is just an executable file at a fixed path Git checks at a fixed
  moment in its own command implementation — there's no registry, no
  config toggling them on generically; Git literally does "if
  `.git/hooks/pre-commit` exists and is executable, run it and check its
  exit code" as a hardcoded step inside `git commit`'s C source. This is
  why renaming away the `.sample` suffix (and `chmod +x`) is the entire
  activation mechanism.
- Hooks run with the **current working directory set to the repo root**
  and receive context either via arguments (`commit-msg` gets the message
  file path) or environment variables (`pre-push` gets remote name/URL on
  stdin, one line per ref being pushed) — this is why a `pre-push` hook
  can inspect exactly which commits are about to leave the machine before
  they do.
- A non-zero exit code from the hook is the *entire* signal Git checks —
  stdout/stderr are just shown to the user for context; Git doesn't parse
  hook output at all, which is why "print an error and `exit 1`" is the
  complete contract any hook author needs to honor.
- `core.hooksPath` works because Git resolves the hooks directory through
  this one config value (defaulting to `$GIT_DIR/hooks`) rather than
  hardcoding `.git/hooks` — pointing it elsewhere is a one-line config
  change with no different mechanism underneath; Git still just looks for
  an executable file with the right name in whatever directory that
  config resolves to.

## Exercise

Write a `pre-commit` hook that rejects any staged file over 1MB (hint:
`git diff --cached --name-only` plus `wc -c` per file), verify it blocks a
deliberately large staged file, then move the working script into a
tracked `.githooks/` directory and configure `core.hooksPath` so it would
work for a fresh clone too.
