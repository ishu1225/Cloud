# GIT — Viva + Exam Notes

---

## WHAT IS GIT

Git is a distributed version control system. Every developer has a full copy of the repository including history. GitHub is a remote hosting platform for Git repos.

---

## CORE CONCEPTS

|Term|Meaning|
|---|---|
|Repository|Folder tracked by Git|
|Working Directory|Where you edit files|
|Staging Area|Files ready to commit (after git add)|
|Commit|Snapshot of staged files saved to history|
|Branch|Independent line of development|
|Remote|Repository hosted on GitHub|
|Clone|Copy a remote repo locally|
|Push|Send local commits to remote|
|Pull|Fetch + merge remote changes locally|
|Merge|Combine two branches|

---

## INITIAL SETUP (one time)

```bash
git config --global user.name "zen1tsu"
git config --global user.email "your@email.com"
git config --list        # verify config
```

---

## SCENARIO 1 — Create new repo and push to GitHub

```bash
# Step 1: Create folder and initialize
mkdir devops-lab
cd devops-lab
git init

# Step 2: Create a file
echo "<h1>DevOps Lab</h1>" > index.html

# Step 3: Stage and commit
git add .
git commit -m "initial commit"

# Step 4: Connect to GitHub (create repo on GitHub first)
git remote add origin https://github.com/zen1tsu/devops-lab.git
git branch -M main
git push -u origin main
```

Verify:

```bash
git log --oneline        # see commits
git remote -v            # see remote URL
git status               # clean working tree
```

---

## SCENARIO 2 — Clone existing repo and make changes

```bash
git clone https://github.com/zen1tsu/devops-lab.git
cd devops-lab

# make a change
echo "updated" >> index.html
git add index.html
git commit -m "update index"
git push origin main
```

---

## SCENARIO 3 — Branching workflow

```bash
# Create and switch to new branch
git checkout -b feature/flask-app

# Make changes
echo "# Flask App" > README.md
git add README.md
git commit -m "add readme"
git push origin feature/flask-app

# Switch back to main and merge
git checkout main
git merge feature/flask-app
git push origin main

# See branch graph
git log --oneline --graph --all
```

---

## SCENARIO 4 — Pull latest changes

```bash
git pull origin main
# same as:
git fetch origin
git merge origin/main
```

---

## SCENARIO 5 — Undo mistakes

```bash
# Unstage a file
git restore --staged index.html

# Discard local changes
git restore index.html

# Undo last commit (keep changes)
git reset --soft HEAD~1

# See what changed
git diff
git diff --staged
```

---

## ALL IMPORTANT COMMANDS

```bash
git init                          # initialize repo
git clone <url>                   # clone remote repo
git status                        # check working tree
git add .                         # stage all files
git add <file>                    # stage specific file
git commit -m "message"           # commit staged files
git push origin main              # push to remote
git pull origin main              # pull from remote
git branch                        # list branches
git branch <name>                 # create branch
git checkout <branch>             # switch branch
git checkout -b <branch>          # create + switch
git merge <branch>                # merge into current
git log --oneline                 # compact commit history
git log --oneline --graph --all   # visual branch graph
git diff                          # see unstaged changes
git remote -v                     # show remote URLs
git remote add origin <url>       # add remote
```

---

## VIVA QUESTIONS

**Q: What is the difference between git fetch and git pull?** A: `git fetch` downloads changes but does not merge. `git pull` does fetch + merge in one step.

**Q: What is the staging area?** A: It is an intermediate area where you prepare files before committing. `git add` moves files to staging, `git commit` saves the snapshot.

**Q: What is the difference between git merge and git rebase?** A: Merge creates a new merge commit preserving branch history. Rebase replays commits on top of another branch making history linear.

**Q: How do you revert a pushed commit?** A: `git revert <commit-hash>` creates a new commit that undoes the changes. Unlike reset it is safe for shared branches.

**Q: What is HEAD in Git?** A: HEAD is a pointer to the current commit you are working on. Normally it points to the latest commit of the current branch.

**Q: What is .gitignore?** A: A file that tells Git which files/folders to not track. Example: `node_modules/`, `*.env`, `__pycache__/`

**Q: Difference between git reset soft, mixed and hard?** A: Soft — undoes commit, keeps files staged. Mixed — undoes commit, unstages files, keeps changes. Hard — undoes commit, discards all changes permanently.

---

## EXAM TIP

If examiner says "show me branching" — do this exact flow:

1. `git checkout -b feature/test`
2. make a change, commit
3. `git push origin feature/test`
4. `git checkout main`
5. `git merge feature/test`
	1. `git log --oneline --graph --all`