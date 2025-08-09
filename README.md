# Git Commands Cheat Sheet & Usage Guide

A single-file reference for the most commonly used Git commands, workflows, and troubleshooting steps.

---

## Prerequisites
- Install [Git for Windows / macOS / Linux]. Use **Git Bash** on Windows for Unix-like commands.
- Configure your identity:
```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
git config --global init.defaultBranch main
````

---

## Quick concepts

* **Working directory** — files you see and edit.
* **Staging area (index)** — files prepared for the next commit (`git add`).
* **Repository (.git)** — where history, refs, and objects are stored.
* **HEAD** — pointer to current commit/branch.

---

## Create a new repository (local)

```bash
# in project folder
git init
```

Link to an existing remote (e.g., GitHub):

```bash
git remote add origin https://github.com/username/repo.git
```

---

## Clone an existing repository

```bash
git clone https://github.com/username/repo.git
# or via SSH
git clone git@github.com:username/repo.git
```

---

## Check repository state

```bash
git status           # what's modified / staged / untracked
git status -s        # short format
```

---

## Stage and commit

```bash
git add file1 file2          # stage specific files
git add .                    # stage all changes and new files (in current directory)
git commit -m "Meaningful message"
# Quick add+commit for tracked files:
git commit -am "Message"     # NOTE: only updates tracked files, doesn't add new untracked files
```

---

## Push to remote

```bash
# First push (sets upstream)
git push -u origin main

# Subsequent pushes
git push
```

---

## Pull from remote

```bash
git pull origin main          # fetch + merge
# Alternatively (safer if you prefer rebasing)
git pull --rebase origin main
```

---

## Fetch vs Pull

```bash
git fetch origin              # update remote-tracking branches (no merge)
git merge origin/main         # merge fetched main into current branch
```

---

## Branching

```bash
git branch                   # list local branches
git branch feature-xyz       # create branch
git checkout feature-xyz     # switch
git checkout -b feature-xyz  # create and switch (shortcut)
git branch -M main           # rename current branch to main
```

---

## Merge & Rebase

```bash
# Merge feature into main
git checkout main
git merge feature-xyz

# Rebase feature onto main (linear history)
git checkout feature-xyz
git rebase main
# Then switch to main and fast-forward
git checkout main
git merge feature-xyz
```

---

## Resolve merge conflicts

When a conflict occurs, files contain markers:

```
<<<<<<< HEAD
your local changes
=======
remote changes
>>>>>>> origin/main
```

Steps:

1. Edit files to resolve conflicts and remove markers.
2. `git add <file>`
3. `git commit -m "Resolve merge conflict in <file>"` (or `git commit` if merging)

---

## Common workflows involving remote-first repos

If remote has a README or initial commit and your local repo started separately:

```bash
# Safely pull and allow unrelated histories (only if necessary)
git pull origin main --allow-unrelated-histories
```

If you want to replace remote completely with local (destructive):

```bash
git push --force origin main
# Warning: this overwrites remote history — use with care
```

---

## Pull a single file from remote (useful for resolving add/add conflicts)

```bash
git fetch origin
git checkout origin/main -- path/to/index.html
# This replaces the local file with the remote version (unstaged)
```

---

## Undoing things (safe → destructive)

```bash
# Unstage a staged file
git reset HEAD file.txt

# Discard changes in working directory (file becomes as in HEAD)
git restore file.txt            # newer safer command
# OR older command:
git checkout -- file.txt

# Reset index and working tree to a commit
git reset --hard <commit>       # WARNING: destroys local changes

# Soft reset (keep changes staged/uncommitted)
git reset --soft <commit>
```

---

## Revert (safe undo of a commit already pushed)

```bash
git revert <commit>            # create new commit that undoes <commit>
```

---

## Stash (save unfinished work)

```bash
git stash                      # stash current changes
git stash list
git stash apply                # reapply most recent stash (keeps stash)
git stash pop                  # reapply and drop stash
git stash drop stash@{0}       # remove a stash
git stash -u                   # stash including untracked files
```

---

## View history & details

```bash
git log                        # full commit log
git log --oneline --graph --decorate --all
git show <commit>
git diff                       # unstaged changes
git diff --staged              # staged changes (what will be committed)
git reflog                     # local reference log for HEAD (useful to recover)
```

---

## Tagging

```bash
git tag v1.0.0
git tag -a v1.0.0 -m "Release 1.0.0"
git push origin --tags
```

---

## Cherry-pick

```bash
git cherry-pick <commit>       # apply a specific commit to your current branch
```

---

## Submodules (brief)

```bash
git submodule add https://github.com/owner/repo.git path/to/submodule
git submodule update --init --recursive
git submodule foreach git pull origin main
```

---

## .gitignore

Create `.gitignore` in repo root to ignore files:

```
# OS
.DS_Store
Thumbs.db

# Node
node_modules/

# Python
__pycache__/
*.pyc

# Editor
.vscode/
.idea/

# Environment
.env
```

If files are already tracked, remove them from the index then update `.gitignore`:

```bash
git rm --cached path/to/file
git commit -m "Remove file from tracking and update .gitignore"
```

---

## Helpful safety & troubleshooting commands

```bash
# Show remote URLs
git remote -v

# Fetch all remotes
git fetch --all

# Show differences between local and remote main
git fetch origin
git log HEAD..origin/main --oneline    # what remote has that local doesn't
git log origin/main..HEAD --oneline    # what local has that remote doesn't

# Recover lost commits
git reflog
git checkout <sha>                     # examine older commit
```

---

## Useful aliases (set once globally)

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg "log --oneline --graph --decorate --all"
```

---

## Common scenarios & commands

### 1. "My push was rejected — remote contains work I don't have"

```bash
git fetch origin
git merge origin/main
# Resolve conflicts if any, commit, then:
git push -u origin main
```

If histories are unrelated (only in special cases), use:

```bash
git pull origin main --allow-unrelated-histories
```

### 2. "I want to take the remote's version of a file and keep my other local files"

```bash
git fetch origin
git checkout origin/main -- path/to/index.html
# edit if needed, then:
git add path/to/index.html style.css
git commit -m "Update index.html and add style.css"
git push -u origin main
```

### 3. "I accidentally ran git init in my home directory"

```bash
# Remove the wrong .git (PowerShell)
Remove-Item -Recurse -Force .git

# or (cmd)
rmdir /s /q .git
```

### 4. "I want to overwrite remote with local (force push)"

```bash
git push --force origin main
# Use only when you understand the consequences (rewrites remote history)
```

---

## Advanced: Rebase interactive & cleaning up history

```bash
# edit the last 3 commits
git rebase -i HEAD~3
# follow interactive instructions to pick/squash/reword
```

---

## Hooks (local automation)

Folder: `.git/hooks/`

* `pre-commit`, `pre-push` — scripts you can add for checks (ex: linting, tests). Make executable.

---

## Best practices / Tips

* Commit early and often with clear messages.
* Use feature branches; keep `main` stable.
* Pull/rebase frequently to minimize conflicts.
* Prefer `pull --rebase` or `rebase` workflow if your team wants a linear history.
* Use `.gitignore` to avoid committing secrets or big files.
* Avoid `--force` unless necessary — prefer `--force-with-lease` which is safer:

```bash
git push --force-with-lease origin main
```

---

## Quick reference: Most-used commands

```bash
git init
git clone <url>
git status
git add .
git commit -m "msg"
git branch -b feature
git checkout feature
git merge feature
git pull --rebase origin main
git push origin main
git log --oneline --graph
```

---

## Resources

* `git help <command>` — offline help for any command (e.g., `git help commit`)
* Use `git config --global` to set defaults and aliases
* Prefer Git Bash on Windows for behavior consistent with most tutorials

---

## Example: Full new-project startup (GitHub)

```bash
# in project folder
git init
echo "# My Project" > README.md
git add README.md
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/username/repo.git
git push -u origin main
```


