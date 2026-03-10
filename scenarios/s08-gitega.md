# Scenario: Gitega - Find the Bad Git Commit

## 🚩 Issue

A Git repository at `/home/admin/git` contains a Golang program and a test. The current HEAD commit fails the test. The task is to find the long hash of the first commit that broke the test.

## 🔍 Investigation

### Step 1: Review commit history
```bash
cd /home/admin/git
git log
```

_Observation: 9 commits total, from `first` to `README.md`. The first commit hash is `9e80a7eb1b09385e93ab4a76cb2c93beec48fd9f`._

### Step 2: Inspect recent commits for clues
```bash
git show f2e018
git show 47995
```

_Observation: The README commit contains a documented solution using `git bisect` and explicitly states: `2e44089778e44dcd9b97aa3baacdcff10311841b is the first bad commit` (message `4th`)_

## ❌ What Didn't Work

- Manual inspection of each commit individually would work but is slow - with more commits this approach doesn't scale.

## ✅ Root Cause

The 4th commit (`2e44089778e44dcd9b97aa3baacdcff10311841b`) introduced a change that broke the test. All subsequent commits inherited this broken state.

## 🛠 Resolution

**The proper automated approach using `git bisect`:**
```bash
# Start bisect session
git bisect start

# Mark current HEAD as bad
git bisect bad HEAD

# Mark the first commit as good
git bisect good 9e80a7eb1b09385e93ab4a76cb2c93beec48fd9f

# Let git automatically run the test on each bisect step
git bisect run go test

# Git outputs the first bad commit hash
# Reset repository to original state
git bisect reset
```

**Save the result:**
```bash
echo "2e44089778e44dcd9b97aa3baacdcff10311841b" > /home/admin/solution
```

## 💡 Lessons Learned

- `git bisect` performs a binary search through commit history - it halves the search space at each step. 9 commits requires only 3-4 steps instead of checking each commit manually.
- `git bisect run <command>` fully automates the process - git runs the command at each step and uses the exit code (0 = good, non-zero = bad) to navigate automatically.
- Always inspect README and documentation files in a repository — they often contain critical context about known issues and solutions.