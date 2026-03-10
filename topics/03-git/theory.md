# Git (Version Control) — Theory Questions

---

## Q1: What is Git and why is it important?

**Answer:**
**Git** is a distributed version control system — it tracks changes to files over time and lets multiple people collaborate on the same project without overwriting each other's work.

**Why it's important in DevOps:**
- Every CI/CD pipeline starts with Git (code change → trigger build)
- Infrastructure as Code files are stored in Git
- Git provides a complete history of every change (who, what, when, why)
- It enables collaboration through branching and merging
- It's the foundation of GitOps (using Git as the single source of truth)

**Key concepts:**
- **Repository (repo)** — A project folder tracked by Git
- **Commit** — A snapshot of your files at a point in time
- **Branch** — A separate line of development
- **Merge** — Combining changes from one branch into another
- **Remote** — A copy of the repo on a server (like GitHub)

---

## Q2: What is the difference between `git merge` and `git rebase`?

**Answer:**
Both combine changes from one branch into another, but they do it differently:

**`git merge`:**
- Creates a new "merge commit" that combines both branches
- Preserves the complete history of both branches
- History looks like a tree with branches

```bash
git checkout main
git merge feature-branch
# Creates a merge commit
```

**`git rebase`:**
- Moves your branch's commits on top of the target branch
- Creates a linear (straight-line) history
- Rewrites commit history (changes commit hashes)

```bash
git checkout feature-branch
git rebase main
# Replays feature commits on top of main
```

**When to use which:**

| Situation | Use |
|-----------|-----|
| Merging a feature branch into main | `merge` (preserves history) |
| Updating your feature branch with latest main | `rebase` (clean history) |
| Public/shared branches | `merge` (don't rewrite shared history) |
| Local/personal branches | `rebase` (cleaner) |

**Golden rule:** Never rebase commits that have been pushed to a shared branch.

---

## Q3: What is a Git conflict and how do you resolve it?

**Answer:**
A **conflict** happens when two people change the same part of the same file, and Git can't automatically decide which change to keep.

**What it looks like:**
```
<<<<<<< HEAD
This is the change on the current branch
=======
This is the change on the other branch
>>>>>>> feature-branch
```

**How to resolve:**
1. Open the file with conflicts
2. Decide which change to keep (or combine both)
3. Remove the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
4. Save the file
5. Stage and commit:
   ```bash
   git add resolved-file.txt
   git commit -m "Resolve merge conflict in resolved-file.txt"
   ```

**Tips to avoid conflicts:**
- Pull frequently to stay up-to-date
- Keep branches short-lived
- Communicate with your team about who's working on what
- Use smaller, focused commits

---

## Q4: What is the difference between `git pull` and `git fetch`?

**Answer:**

- **`git fetch`** — Downloads changes from the remote but does NOT apply them to your working files. It updates your local knowledge of what's on the remote.
- **`git pull`** — Does `git fetch` + `git merge` in one step. Downloads AND applies changes.

```bash
# Fetch only (safe — doesn't change your files)
git fetch origin
git log origin/main    # See what changed on remote
git merge origin/main  # Apply when ready

# Pull (fetch + merge in one step)
git pull origin main
```

**Best practice:** Use `git fetch` first to review changes before merging, especially on important branches.

---

## Q5: What are the common Git branching strategies?

**Answer:**

**1. Git Flow:**
- `main` — Production-ready code
- `develop` — Integration branch for features
- `feature/*` — Individual features
- `release/*` — Preparing for a release
- `hotfix/*` — Emergency fixes for production
- Best for: Projects with scheduled releases

**2. GitHub Flow (simpler):**
- `main` — Always deployable
- `feature/*` — Branch off main, merge back via pull request
- Best for: Continuous deployment, web applications

**3. Trunk-Based Development:**
- Everyone commits to `main` (trunk) frequently
- Short-lived feature branches (< 1 day)
- Uses feature flags to hide incomplete features
- Best for: High-performing teams, CI/CD-heavy environments

**Which to choose:**

| Strategy | Team Size | Release Frequency | Complexity |
|----------|-----------|-------------------|------------|
| Git Flow | Large | Scheduled | High |
| GitHub Flow | Any | Continuous | Low |
| Trunk-Based | Experienced | Continuous | Low |

---

## Q6: What is a pull request (PR) / merge request (MR)?

**Answer:**
A **pull request** (GitHub/Bitbucket term) or **merge request** (GitLab term) is a request to merge your branch's changes into another branch. It's a collaboration tool that enables:

- **Code review** — Team members review your changes before merging
- **Discussion** — Comments and suggestions on specific lines
- **CI/CD checks** — Automated tests run on the proposed changes
- **Approval workflow** — Required approvals before merging

**Good PR practices:**
- Keep PRs small and focused (one feature or fix per PR)
- Write a clear description of what changed and why
- Include screenshots for UI changes
- Link to the related issue or ticket
- Respond to review comments promptly

---

## Q7: What is `.gitignore` and why is it important?

**Answer:**
`.gitignore` is a file that tells Git which files and folders to ignore (not track). This keeps your repository clean and prevents sensitive or unnecessary files from being committed.

**Common entries:**
```gitignore
# Dependencies
node_modules/
vendor/
venv/

# Build output
dist/
build/
*.o
*.class

# Environment and secrets
.env
*.pem
*.key

# IDE files
.vscode/
.idea/
*.swp

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Terraform
*.tfstate
*.tfstate.backup
.terraform/
```

**Why it's important:**
- Prevents secrets (API keys, passwords) from being committed
- Keeps the repo small (no `node_modules` or build artifacts)
- Avoids merge conflicts on auto-generated files

**If you already committed a file you want to ignore:**
```bash
# Remove from Git tracking but keep the file locally
git rm --cached secret-file.env
echo "secret-file.env" >> .gitignore
git commit -m "Stop tracking secret-file.env"
```

---

## Q8: What is `git stash`?

**Answer:**
`git stash` temporarily saves your uncommitted changes so you can work on something else, then come back to them later.

```bash
# Save current changes
git stash
# or with a message
git stash save "work in progress on login feature"

# List all stashes
git stash list

# Apply the most recent stash (keeps it in the stash list)
git stash apply

# Apply and remove from stash list
git stash pop

# Apply a specific stash
git stash apply stash@{2}

# Delete a stash
git stash drop stash@{0}

# Clear all stashes
git stash clear
```

**Common use case:** You're working on a feature, but need to quickly fix a bug on another branch. Stash your changes, fix the bug, then pop your stash to continue.

---

## Q9: What is `git cherry-pick`?

**Answer:**
`git cherry-pick` takes a specific commit from one branch and applies it to another branch. Unlike merge (which brings all commits), cherry-pick selects individual commits.

```bash
# Apply a specific commit to current branch
git cherry-pick abc1234

# Cherry-pick without committing (just stage the changes)
git cherry-pick --no-commit abc1234

# Cherry-pick a range of commits
git cherry-pick abc1234..def5678
```

**Use cases:**
- Apply a bug fix from `develop` to `main` without merging everything
- Port a specific feature to a release branch
- Recover a commit from a deleted branch

---

## Q10: What is `git reset` vs `git revert`?

**Answer:**

**`git reset`** — Moves the branch pointer backward, effectively "undoing" commits. Changes commit history.
```bash
# Soft reset — undo commit but keep changes staged
git reset --soft HEAD~1

# Mixed reset (default) — undo commit and unstage changes
git reset HEAD~1

# Hard reset — undo commit and DELETE all changes
git reset --hard HEAD~1    # ⚠️ Destructive!
```

**`git revert`** — Creates a NEW commit that undoes a previous commit. Does NOT change history.
```bash
git revert abc1234
# Creates a new commit that reverses the changes in abc1234
```

| Feature | `git reset` | `git revert` |
|---------|------------|-------------|
| Changes history? | Yes | No (adds new commit) |
| Safe for shared branches? | No | Yes |
| Use case | Undo local commits | Undo public commits |

**Rule:** Use `revert` for commits already pushed to a shared branch. Use `reset` only for local commits.

---

## Q11: What are Git hooks?

**Answer:**
**Git hooks** are scripts that run automatically at certain points in the Git workflow. They let you automate tasks like code formatting, running tests, or checking commit messages.

**Common hooks:**

| Hook | When It Runs | Use Case |
|------|-------------|----------|
| `pre-commit` | Before a commit is created | Lint code, run formatters |
| `commit-msg` | After writing commit message | Enforce commit message format |
| `pre-push` | Before pushing to remote | Run tests |
| `post-merge` | After a merge | Install dependencies |
| `pre-receive` | Server-side, before accepting push | Enforce policies |

**Example pre-commit hook:**
```bash
#!/bin/bash
# .git/hooks/pre-commit

# Run linter
npm run lint
if [ $? -ne 0 ]; then
    echo "Linting failed. Fix errors before committing."
    exit 1
fi
```

**Tools for managing hooks:**
- **Husky** — Popular for Node.js projects
- **pre-commit** — Python-based, language-agnostic
- **lefthook** — Fast, written in Go

---

## Q12: What is `git bisect`?

**Answer:**
`git bisect` helps you find which commit introduced a bug by doing a binary search through your commit history. Instead of checking every commit, it narrows it down efficiently.

```bash
# Start bisecting
git bisect start

# Mark the current (broken) commit as bad
git bisect bad

# Mark a known good commit
git bisect good abc1234

# Git checks out a commit in the middle — test it
# If it's good:
git bisect good
# If it's bad:
git bisect bad

# Repeat until Git finds the exact commit
# Git will say: "abc5678 is the first bad commit"

# When done, return to your original branch
git bisect reset
```

**Automated bisect:**
```bash
# Run a test script automatically
git bisect start HEAD abc1234
git bisect run ./test-script.sh
# Git will automatically find the bad commit
```

---

## Q13: What are Git submodules?

**Answer:**
**Git submodules** let you include one Git repository inside another. The parent repo tracks which commit of the submodule to use.

```bash
# Add a submodule
git submodule add https://github.com/example/library.git libs/library

# Clone a repo with submodules
git clone --recurse-submodules https://github.com/example/project.git

# Update submodules
git submodule update --init --recursive

# Pull latest changes in submodules
git submodule update --remote
```

**Use cases:**
- Shared libraries used across multiple projects
- Third-party dependencies you want to pin to a specific version
- Monorepo-like structure with independent components

**Alternatives:** Git subtree, package managers (npm, pip), or monorepo tools (Nx, Turborepo).

---

## Q14: What is the difference between `git clone`, `git fork`, and `git mirror`?

**Answer:**

- **`git clone`** — Creates a local copy of a remote repository. You can push changes back if you have permission.
  ```bash
  git clone https://github.com/user/repo.git
  ```

- **`git fork`** — Creates a copy of someone else's repository under YOUR account (GitHub/GitLab feature, not a Git command). Used for contributing to open-source projects.
  - Fork → Clone → Make changes → Push to your fork → Create a pull request

- **`git clone --mirror`** — Creates an exact copy of a repository, including all branches, tags, and refs. Used for backups or migrating repos.
  ```bash
  git clone --mirror https://github.com/user/repo.git
  ```

---

## Q15: How do you write good commit messages?

**Answer:**

**Format (Conventional Commits):**
```
<type>(<scope>): <short description>

<optional longer description>

<optional footer>
```

**Types:**
- `feat` — New feature
- `fix` — Bug fix
- `docs` — Documentation changes
- `style` — Formatting (no code change)
- `refactor` — Code restructuring (no feature/fix)
- `test` — Adding or updating tests
- `chore` — Maintenance tasks (dependencies, configs)
- `ci` — CI/CD changes

**Examples:**
```
feat(auth): add OAuth2 login with Google

fix(api): handle null response from payment service

docs(readme): update installation instructions for v3

chore(deps): upgrade Terraform provider to 5.0
```

**Rules:**
- Use imperative mood ("add" not "added")
- Keep the first line under 72 characters
- Explain WHY, not just WHAT
- Reference issue numbers when applicable