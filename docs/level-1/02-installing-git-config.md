# 02 · Installing Git & Configuration

## Installing Git

```bash
# macOS (Homebrew)
brew install git

# macOS also ships Git via Xcode Command Line Tools:
xcode-select --install

# Ubuntu/Debian
sudo apt update && sudo apt install git

# Fedora
sudo dnf install git

# Windows: download the installer from https://git-scm.com/download/win
# (this also installs "Git Bash", a Unix-like shell many Windows users prefer)
```

Verify the install:

```bash
git --version
# git version 2.5x.x
```

Any recent version (2.3x+) works fine for this course.

## Telling Git who you are

Every commit records an author name and email. Git won't let you commit
until these are set. Configure them once, globally, and they apply to every
repository on your machine:

```bash
git config --global user.name "Ada Lovelace"
git config --global user.email "ada@example.com"
```

Use the same email you'll use for your GitHub account later — GitHub
matches commit authorship to profiles by email, which is how commits show
up correctly attributed to you on your GitHub profile.

Check what's set:

```bash
git config --global user.name
# Ada Lovelace

git config --list
# user.name=Ada Lovelace
# user.email=ada@example.com
# ... (plus other settings)
```

## `--global` vs local config

- `git config --global ...` writes to `~/.gitconfig` — applies to every
  repository for your user account.
- `git config ...` (no `--global`, run inside a repo) writes to
  `.git/config` in that repository only — useful if, say, your work
  projects need a different email than your personal ones.
- Local config always overrides global config for that one repository.

```bash
# Inside a specific repo, override just for that repo:
cd ~/projects/work-repo
git config user.email "ada@company.com"
```

## Useful config defaults worth setting

```bash
# Name the default branch "main" for every new repo you create
git config --global init.defaultBranch main

# Pick your default editor for commit messages (nano is beginner-friendly)
git config --global core.editor "nano"
# Or, if you use VS Code:
git config --global core.editor "code --wait"

# Make colored output the default (usually already on)
git config --global color.ui auto

# Store credentials so you're not typing a password on every push
# (macOS keeps this in Keychain automatically; on Linux/Windows:)
git config --global credential.helper cache
```

## Where config lives

There are three levels, in increasing priority:

| Level | Flag | File |
|---|---|---|
| System | `--system` | `/etc/gitconfig` (all users on the machine) |
| Global | `--global` | `~/.gitconfig` (your user account) |
| Local | *(none)* | `.git/config` (one specific repository) |

```bash
# See exactly which file a setting came from
git config --show-origin user.name
```

## Getting help

Git's built-in help is thorough and always matches the version installed:

```bash
git help commit        # opens the full manual page
git commit --help      # same thing
git commit -h          # quick usage summary
```

## Worked example: a fresh machine, end to end

```bash
$ git --version
git version 2.43.0

$ git config --global user.name "Ada Lovelace"
$ git config --global user.email "ada@example.com"
$ git config --global init.defaultBranch main

$ git config --list --global
user.name=Ada Lovelace
user.email=ada@example.com
init.defaultbranch=main
```

That's the entire one-time setup — from here on, every `git init` on this
machine creates a `main` branch by default, and every commit is correctly
attributed to Ada.

## Exercise

Install Git if you haven't already, then set your global `user.name` and
`user.email` to your own details (use the email you plan to sign up to
GitHub with in module 7). Set `init.defaultBranch` to `main`. Finally, run
`git config --list --global` and confirm all three values appear correctly.
