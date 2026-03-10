# Git — Scenario-Based Questions

---

## S1: You accidentally committed a file with sensitive data (like an API key) and pushed it. What do you do?

**Answer:**

**Immediate steps:**

1. **Revoke the exposed secret immediately** — Change the API key, password, or token. This is the most important step because even if you remove it from Git, it may already be cached or cloned.

2. **Remove the file from the latest commit:**
   ```bash
   # If it was the last commit and not yet pushed
   git reset --soft HEAD~1
   # Remove the file
   git rm --cached secret-file.env
   echo "secret-file.env" >> .gitignore
   git commit -m "Remove sensitive file"
   ```

3. **If already pushed, rewrite history:**
   ```bash
   # Using git-filter-repo (recommended tool)
   pip install git-filter-repo
   git filter-repo --path secret-file.env --invert-paths
   
   # Force push (⚠️ coordinate with your team first)
   git push --force-with-lease
   ```

4. **Notify your team:**
   - Tell everyone to re-clone or rebase
   - Report the exposure to your security team
   - Check audit logs for unauthorized access

5. **Prevent it from happening again:**
   - Add the file to `.gitignore`
   - Use pre-commit hooks to scan for secrets (tools: `gitleaks`, `trufflehog`)
   - Use environment variables or a secrets manager instead of files
   - Enable GitHub's secret scanning feature

---

## S2: Two developers are working on the same file and both push changes. How do you resolve this?

**Answer:**

1. **The second developer will get a rejection when pushing:**
   ```
   ! [rejected] main -> main (fetch first)
   ```

2. **Pull the latest changes:**
   ```bash
   git pull origin main
   # Git will try to auto-merge
   ```

3. **If there's a conflict:**
   ```bash
   # Git shows which files have conflicts
   git status
   
   # Open the conflicted file — you'll see:
   # <<<<<<< HEAD
   # Developer A's changes
   # =======
   # Developer B's changes
   # >>>>>>> origin/main
   ```

4. **Resolve the conflict:**
   - Decide which changes to keep (or combine both)
   - Remove the conflict markers
   - Test the code to make sure it works

5. **Complete the merge:**
   ```bash
   git add conflicted-file.txt
   git commit -m "Resolve merge conflict in conflicted-file.txt"
   git push origin main
   ```

**Prevention:**
- Use feature branches (each developer works on their own branch)
- Keep branches short-lived and merge frequently
- Use pull requests for code review before merging
- Communicate about who's working on which files

---

## S3: You need to undo the last 3 commits on a shared branch without losing the changes. How?

**Answer:**

Since it's a shared branch, use `git revert` (not `git reset`) to avoid rewriting history:

```bash
# Revert the last 3 commits (creates 3 new revert commits)
git revert HEAD~2..HEAD

# Or revert them one by one
git revert HEAD        # Revert most recent
git revert HEAD~1      # Revert second most recent  
git revert HEAD~2      # Revert third most recent

# Or revert all 3 in a single commit
git revert --no-commit HEAD~2..HEAD
git commit -m "Revert last 3 commits due to deployment issue"

# Push the revert commits
git push origin main
```

**Why not `git reset`?**
- `git reset` rewrites history, which breaks things for everyone who already pulled those commits
- `git revert` adds new commits that undo the changes, keeping history intact

---

## S4: Your team's Git repository has grown very large (several GB). How do you investigate and fix this?

**Answer:**

1. **Find what's taking up space:**
   ```bash
   # Check repo size
   git count-objects -vH
   
   # Find the largest files in history
   git rev-list --objects --all | \
     git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
     sort -k3 -n -r | head -20
   ```

2. **Common causes:**
   - Large binary files (videos, images, datasets) committed accidentally
   - Build artifacts committed to the repo
   - Dependency folders (node_modules, vendor) committed

3. **Remove large files from history:**
   ```bash
   # Using git-filter-repo
   git filter-repo --strip-blobs-bigger-than 10M
   
   # Or remove specific files
   git filter-repo --path large-video.mp4 --invert-paths
   ```

4. **Prevent future issues:**
   - Use **Git LFS (Large File Storage)** for large files:
     ```bash
     git lfs install
     git lfs track "*.psd"
     git lfs track "*.zip"
     git add .gitattributes
     ```
   - Update `.gitignore` to exclude build artifacts and dependencies
   - Set up pre-commit hooks to reject large files
   - Add a repo size check in CI/CD

5. **After cleanup:**
   ```bash
   # Garbage collect
   git gc --aggressive --prune=now
   
   # Force push (coordinate with team)
   git push --force-with-lease
   
   # Team members should re-clone
   ```

---

## S5: You need to set up a Git workflow for a team of 10 developers working on a web application with weekly releases. What do you recommend?

**Answer:**

**Recommended: GitHub Flow with release branches**

```
main (always deployable)
  ├── feature/user-auth (developer A)
  ├── feature/search-api (developer B)
  ├── fix/login-bug (developer C)
  └── release/v2.5 (when preparing a release)
```

**Workflow:**

1. **Branch naming convention:**
   - `feature/description` — New features
   - `fix/description` — Bug fixes
   - `release/vX.Y` — Release preparation
   - `hotfix/description` — Emergency production fixes

2. **Development process:**
   ```bash
   # Start a new feature
   git checkout main
   git pull origin main
   git checkout -b feature/user-auth
   
   # Work on the feature, commit frequently
   git add .
   git commit -m "feat(auth): add login form"
   
   # Push and create a pull request
   git push origin feature/user-auth
   # Create PR on GitHub → request review
   ```

3. **Pull request rules:**
   - At least 1 approval required
   - All CI checks must pass (lint, tests, build)
   - Branch must be up-to-date with main
   - Squash merge to keep history clean

4. **Release process:**
   ```bash
   # Create release branch from main
   git checkout -b release/v2.5 main
   # Only bug fixes go into the release branch
   # When ready, merge to main and tag
   git tag v2.5
   ```

5. **Branch protection rules (GitHub):**
   - Require pull request reviews
   - Require status checks to pass
   - No direct pushes to main
   - Require linear history (squash merges)

---

## S6: A developer says "I lost my commits!" after a bad rebase. How do you help them recover?

**Answer:**

**Git rarely loses data permanently.** Use `git reflog` to find the lost commits:

```bash
# Show the reflog — a log of every HEAD position
git reflog

# Output looks like:
# abc1234 HEAD@{0}: rebase: current
# def5678 HEAD@{1}: rebase: start
# 789abcd HEAD@{2}: commit: my important work  ← This is what we want!
# ...

# Option 1: Reset to the commit before the rebase
git reset --hard 789abcd

# Option 2: Create a new branch from the lost commit
git checkout -b recovered-work 789abcd

# Option 3: Cherry-pick specific lost commits
git cherry-pick 789abcd
```

**Important notes:**
- `git reflog` only works locally (not on remote)
- Reflog entries expire after 90 days by default
- This is why `git reflog` is called "the safety net"

**Prevention:**
- Before risky operations, create a backup branch:
  ```bash
  git branch backup-before-rebase
  git rebase main
  # If something goes wrong:
  git reset --hard backup-before-rebase
  ```