---
description: "Creating a GitHub Repo & Pushing — There are two ways to connect a project to GitHub: create the GitHub repo first and clone it, or start locally and…"
---

# 08 · Creating a GitHub Repo & Pushing

There are two ways to connect a project to GitHub: create the GitHub repo
first and clone it, or start locally and connect a GitHub repo afterward.
Both are common; this module walks through both.

## Method 1: create on GitHub, then clone

1. On GitHub, click the **+** icon (top-right) → **New repository**.
2. Give it a name (e.g. `hello-git`), an optional description, choose
   **Public** or **Private**, and — importantly, for this first method —
   check **Add a README file**. This creates an initial commit so the repo
   isn't empty.
3. Click **Create repository**.
4. On the new repo's page, click **`<> Code`** and copy the HTTPS (or SSH)
   URL.

```bash
git clone https://github.com/yourname/hello-git.git
cd hello-git
ls
# README.md
```

You now have a fully working local clone, already wired up with `origin`
pointing at GitHub:

```bash
git remote -v
# origin  https://github.com/yourname/hello-git.git (fetch)
# origin  https://github.com/yourname/hello-git.git (push)
```

## Method 2: start locally, connect to GitHub after

If you already have a local project (like the practice repos from earlier
modules) and want to push it up:

1. On GitHub, click **+** → **New repository**. Give it a name, but this
   time **leave "Add a README" unchecked** — you already have local files,
   and you don't want GitHub creating a commit that immediately conflicts
   with your local history.
2. Click **Create repository**. GitHub shows you the exact commands to run
   — they'll look like this:

```bash
cd your-existing-project
git init -b main                                        # if not already a repo
git add -A
git commit -m "Initial commit"                          # if not already committed
git remote add origin https://github.com/yourname/your-existing-project.git
git push -u origin main
```

## Authenticating the push

The first `git push` will prompt for credentials. As covered in the
previous module, GitHub requires a **Personal Access Token** in place of a
password for HTTPS, or an SSH key. Setting up a token:

1. GitHub → your profile picture → **Settings** → **Developer settings** →
   **Personal access tokens** → **Tokens (classic)** → **Generate new
   token**.
2. Select the `repo` scope (full control of repositories), set an
   expiration, and generate. **Copy the token immediately** — GitHub only
   shows it once.
3. When Git prompts for a password during `git push`, paste the token
   instead. Most credential managers (macOS Keychain, Git Credential
   Manager on Windows) then remember it for future pushes.

Alternatively, authenticate once via the `gh` CLI and let it manage tokens
for you:

```bash
gh auth login
# Follow the interactive prompts (choose GitHub.com, HTTPS, login via browser)
```

## Confirming the push worked

```bash
git push -u origin main
```
```
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 230 bytes | 230.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/yourname/hello-git.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

Refresh the GitHub repo page in your browser — your files and commit
history should now be visible there.

## Pushing subsequent changes

Once `-u` has linked the branch, every future push/pull on `main` is just:

```bash
git add -A
git commit -m "Add a new feature"
git push
```

## A note on repo visibility

- **Public** repos are visible to anyone on the internet — good for
  portfolios, open-source projects, and this course's exercises.
- **Private** repos are visible only to you and people you explicitly
  invite as collaborators — good for anything you're not ready to share,
  including work code (check your employer's policies before pushing any
  work-related code anywhere, public or private).

You can change visibility later from **Settings → General → Danger Zone**
on the repo page.

## Worked example: full walkthrough

```bash
mkdir hello-git-demo && cd hello-git-demo
git init -b main
git config user.name "Ada Lovelace"
git config user.email "ada@example.com"

echo "# Hello Git Demo" > README.md
git add README.md
git commit -m "Initial commit"

# after creating an empty repo named hello-git-demo on GitHub:
git remote add origin https://github.com/adalovelace/hello-git-demo.git
git push -u origin main
```

If this is your first push ever from this machine, you'll be prompted to
authenticate (token or `gh auth login`) at this step.

## How It Actually Works

The `-u` (`--set-upstream`) flag and the "branch not tracking" errors you'll
occasionally see both come from one small piece of metadata:

- `git push -u origin main` does two separate things: it pushes your
  objects and updates `origin`'s `refs/heads/main`, *and* it writes two
  lines into `.git/config` — `branch.main.remote = origin` and
  `branch.main.merge = refs/heads/main`. That's the entire "tracking
  relationship." Every bare `git push`/`git pull` afterward reads those two
  lines to figure out which remote and which remote branch to talk to,
  which is why you can drop `origin main` from every subsequent command.
- Your local commit objects are byte-for-byte the same objects that exist
  on GitHub after a push — the SHA of a commit doesn't change when you push
  it. This is why `git push` can be so fast the second time: Git compares
  the SHAs it has against the SHAs the server reports having, and only
  transmits objects the server is actually missing, packed and
  delta-compressed together into one packfile for the connection.
- Repo **visibility** (public/private) and features like the web file
  browser exist entirely in GitHub's application layer, sitting in front of
  the same bare repository — flipping a repo from private to public
  doesn't touch a single Git object; it only changes an access-control
  check GitHub's servers apply before serving requests for that
  repository's data.
- Creating an *empty* repo on GitHub (no README) matters mechanically: if
  you'd let GitHub initialize the repo with a README, GitHub would have
  created an initial commit server-side with no shared ancestor with your
  local history, and your first `git push` would be rejected as
  non-fast-forward (the same rejection mechanism from the previous
  module) — you'd have had to `pull` and merge the unrelated histories
  first.

## Exercise

Create a new, empty GitHub repository (no README) called `git-course-demo`.
Locally, create a folder with the same name, `git init` it, add a file, and
commit. Connect it to the GitHub repo with `git remote add origin ...` and
push with `git push -u origin main`. Confirm your commit shows up on
GitHub by viewing the repo in your browser.
