# Git - Version Control & Repository Diagnostics

Essential commands for navigating repositories, inspecting history, and finding breaking changes during incident investigation.

## 1. Repository Status & History
* `git status` - Show changed, staged, and untracked files in the working directory.
* `git log` - List all commits with hash, author, date, and message.
* `git log --oneline` - Compact view: one commit per line with short hash.
* `git log --oneline --graph` - Visual branch history as ASCII graph.
* `git show <hash>` - Show full details and diff of a specific commit.

## 2. Comparing Changes
* `git diff` - Show unstaged changes in the working directory.
* `git diff <hash1> <hash2>` - Compare changes between two commits.
* `git diff <hash> -- <file>` - Compare a specific file between a commit and current state.

## 3. Bisect - Finding a Bad Commit
Used to find the first commit that introduced a bug using binary search:
```bash
# Start bisect session
git bisect start

# Mark current commit as bad
git bisect bad HEAD

# Mark a known good commit (e.g. first commit)
git bisect good <hash>

# Automated mode: run a test at each step
git bisect run <test-command>

# Git outputs the first bad commit hash
# Always reset after finishing
git bisect reset
```

## 4. Branches & Navigation
* `git branch` - List all local branches.
* `git branch -a` - List all local and remote branches.
* `git checkout <branch>` - Switch to an existing branch.
* `git checkout <hash>` - Inspect repository state at a specific commit (detached HEAD).
* `git checkout -b <name>` - Create and switch to a new branch.

## 5. Remote & Sync
* `git pull` - Fetch and merge changes from remote repository.
* `git push origin <branch>` - Push committed changes to remote branch.
* `git fetch` - Download remote changes without merging.

## 6. Tags & Releases
* `git tag` - List all tags.
* `git tag <tag-name>` - Create a lightweight tag on current commit.
* `git show <tag-name>` - Show details of a tagged commit.