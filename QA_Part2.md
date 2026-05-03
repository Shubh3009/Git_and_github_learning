# 🧪 Git & GitHub — Practice Questions (Part 2 of 2)
> Covers: Stash · Reset · Rebase · Remote/GitHub · SSH · Config · Open Source

---

## 📦 SECTION 5 — Git Stash

---

### Q23. What problem does `git stash` solve? Give a real scenario.

**Answer:**
`git stash` solves the problem of needing to switch branches when you have **uncommitted work-in-progress**.

**Scenario from your session:**
You're on `footer` branch, halfway through editing `index.html`:
```html
<body>
    footer was added successfully
    update index file with footer code    ← WIP, not ready to commit
</body>
```
Your manager says: "Fix a bug on `bugfix` branch NOW!"
You can't switch with uncommitted changes (Git will warn or refuse). Solution:

```bash
git stash           # saves WIP, reverts index.html to last commit state
git checkout bugfix # now you can switch cleanly
# ... fix the bug ...
git checkout footer
git stash pop       # WIP line is restored!
```

---

### Q24. What is the difference between `git stash pop` and `git stash apply`?

**Answer:**

| Command | Applies stash? | Removes from stack? |
|---------|---------------|---------------------|
| `git stash pop` | ✅ Yes | ✅ Yes |
| `git stash apply` | ✅ Yes | ❌ No (stays in stash list) |

```bash
git stash list
# stash@{0}: WIP on footer: 4dd2a6d adding readme.md

git stash apply         # apply but keep it
git stash list
# stash@{0}: WIP on footer: ...   ← still here

git stash pop           # apply AND remove
git stash list
# (empty)
```

Use `apply` when you want to apply the same stash to multiple branches.

---

### Q25. Can you apply a stash on a DIFFERENT branch than where you stashed it?

**Answer:**
**Yes!** Stash is global to the repo — not tied to a branch.

```bash
# On footer branch — stash changes
git stash

# Switch to bugfix branch
git checkout bugfix

# Apply footer's stash here
git stash pop
# Your footer edits are now on bugfix branch
```

This is useful when you realise you started work on the wrong branch.

---

### Q26. You stashed 3 times. Show what `git stash list` outputs and explain the structure.

**Answer:**
```bash
git stash list
```
```
stash@{0}: WIP on footer: 4dd2a6d adding readme.md     ← NEWEST
stash@{1}: WIP on main:   5628807 changed index and footer
stash@{2}: WIP on bugfix: a1b2c3d initial               ← OLDEST
```

It's a **stack** — newest is `stash@{0}`. `git stash pop` always takes `stash@{0}`.

To apply a specific stash:
```bash
git stash apply stash@{2}   # apply the oldest one
git stash drop stash@{1}    # delete stash@{1}
git stash clear             # delete ALL stashes
```

---

## ↩️ SECTION 6 — Git Reset

---

### Q27. What are the 3 types of `git reset`? Compare what each does to HEAD, Index, and Working Directory.

**Answer:**

Starting state: `C1 → C2 → C3 (HEAD)`

| Reset Type | HEAD moves? | Index (staged)? | Working Dir files? |
|-----------|------------|----------------|-------------------|
| `--soft HEAD~1` | ✅ Back to C2 | Keeps C3 changes staged | ❌ Unchanged |
| `--mixed HEAD~1` (default) | ✅ Back to C2 | ❌ C3 changes unstaged | ❌ Unchanged |
| `--hard HEAD~1` | ✅ Back to C2 | ❌ C3 changes gone | ✅ Reverted to C2 |

```bash
git reset --soft HEAD~1    # undo commit, keep files staged
git reset HEAD~1           # undo commit, unstage files (changes preserved)
git reset --hard HEAD~1    # undo commit, DELETE all changes ⚠️
```

---

### Q28. When would you use `git reset --soft` vs `git reset --hard`?

**Answer:**

**Use `--soft` when:** You committed too early and want to add more files or rewrite the message — your changes are safe and staged:
```bash
git commit -m "oops wrong message"
git reset --soft HEAD~1         # undo commit, files stay staged
git commit -m "correct message" # recommit with right message
```

**Use `--hard` when:** You want to completely throw away the last commit AND its changes — dangerous but necessary for bad commits:
```bash
git commit -am "added about us section"   # from your session
# Whoops! Don't want this at all
git reset --hard HEAD~1
# That commit and all its changes are gone forever
```

---

### Q29. What is `HEAD~1`, `HEAD~2`? What does `HEAD~0` mean?

**Answer:**

| Expression | Meaning |
|------------|---------|
| `HEAD` or `HEAD~0` | Current commit |
| `HEAD~1` | One commit before current |
| `HEAD~2` | Two commits before current |
| `4dd2a6d` | Jump directly to that commit hash |

```bash
git log --oneline
# C3  added footer   ← HEAD
# C2  add navbar
# C1  add index

git reset --hard HEAD~2   # jumps back to C1, C2 and C3 are gone!
```

---

### Q30. What is a "Detached HEAD" state? How do you get into it and get out?

**Answer:**
Normally HEAD points to a **branch** which points to a commit. In detached HEAD, HEAD points **directly to a commit** — not a branch.

**Getting in:**
```bash
git checkout 4dd2a6d    # checkout a specific commit hash
```
```
cat .git/HEAD
# 4dd2a6d...  ← raw hash, not "ref: refs/heads/main"
```

**Warning shown:**
```
You are in 'detached HEAD' state. Any commits made here can be lost.
```

**Getting out:**
```bash
git checkout main       # reattach to a branch
cat .git/HEAD
# ref: refs/heads/main  ← normal again
```

> ⚠️ Any commits made in detached HEAD will be lost (orphaned) unless you create a branch: `git checkout -b save-my-work`

---

## 🔁 SECTION 7 — Git Rebase

---

### Q31. Explain what `git rebase main` does when run on `bugfix` branch. Show the before and after diagram.

**Answer:**
Rebase picks up `bugfix`'s commits and **replays them on top of main's latest commit**.

**Before:**
```
main:    C1 ──► C2 (navbar) ──► C3 (pricing card)
                  \
bugfix:            C4 (about us fixed)
```

```bash
git checkout bugfix
git rebase main
```

**After:**
```
main:    C1 ──► C2 ──► C3 (pricing card)
                              \
bugfix:                        C4' (about us fixed) ← replayed!
```

`C4'` is a **new commit** (different SHA) with the same changes as `C4`.

---

### Q32. How does Git perform a rebase internally? (4 steps)

**Answer:**
```
git rebase main   (while on bugfix)
```

1. **Find common ancestor** of `bugfix` and `main` → `C2`
2. **Save** all `bugfix` commits after `C2` → temporarily stores `C4`
3. **Reset** `bugfix` pointer to match `main` tip → `C3`
4. **Re-apply** saved commits one by one on top → `C4` becomes `C4'` (new SHA)

Result: Linear history, as if you always started from `main`'s latest.

---

### Q33. A conflict occurs during rebase. Show the exact steps to resolve it and continue.

**Answer:**
```bash
git rebase main
# CONFLICT (content): Merge conflict in index.html
# error: could not apply a1b2c3d... card added to index file
```

`index.html` shows:
```html
<<<<<<< HEAD
    images are being added to the project    ← from main
=======
    card added to index file                 ← from bugfix
>>>>>>> a1b2c3d
```

**Resolution:**
```bash
# Step 1: Edit index.html — remove markers, keep both lines
# Step 2: Stage the file
git add index.html
# Step 3: Continue rebase (NOT git commit!)
git rebase --continue
# Output: Successfully rebased and updated refs/heads/bugfix.
```

To abort and go back to before the rebase:
```bash
git rebase --abort
```

---

### Q34. What is the "Golden Rule of Rebase"? Why?

**Answer:**

> **Never rebase a branch that has been pushed to a shared remote.**

**Why:** Rebase rewrites history — each replayed commit gets a **new SHA hash**. If your teammate already cloned the old hashes and you push the rebased history, their Git sees them as completely different commits → broken, conflicting history.

```
✅ Safe:   Rebase your LOCAL feature branch before opening a PR
❌ Unsafe: Rebase main, develop, or any branch others have pulled
```

---

### Q35. Merge vs Rebase — fill in the comparison table.

**Answer:**

| Feature | Merge | Rebase |
|---------|-------|--------|
| History shape | Non-linear (has merge commits) | Linear (clean) |
| Creates new commit? | ✅ Yes (merge commit) | ❌ No (replays existing commits) |
| Safe on shared branches? | ✅ Yes | ❌ No |
| Conflict resolution | Once (in merge commit) | Once per replayed commit |
| Best used for | Integrating finished features | Cleaning up before a PR |

---

## 🌐 SECTION 8 — Remote, GitHub, Push, Pull, Clone

---

### Q36. What does `git remote add origin <url>` do? Does it upload anything?

**Answer:**
It registers a remote server address with the nickname `origin` in `.git/config`. **It does NOT upload anything.**

```bash
git remote add origin https://github.com/Shubh3009/Rough_github.git
```

Adds to `.git/config`:
```
[remote "origin"]
    url = https://github.com/Shubh3009/Rough_github.git
    fetch = +refs/heads/*:refs/remotes/origin/*
```

```bash
git remote -v
# origin  https://github.com/Shubh3009/Rough_github.git (fetch)
# origin  https://github.com/Shubh3009/Rough_github.git (push)
```

---

### Q37. What does the `-u` flag in `git push -u origin main` do?

**Answer:**
`-u` = `--set-upstream`. It links your local `main` branch to `origin/main` (the remote tracking branch).

```bash
git push -u origin main   # first push — sets upstream
```

**Effect:** All future pushes and pulls become shorter:
```bash
git push    # no need for "origin main" anymore!
git pull    # same — knows where to pull from
```

Without `-u`, you'd always need: `git push origin main`.

---

### Q38. What is the difference between `git fetch` and `git pull`?

**Answer:**

| | `git fetch` | `git pull` |
|--|------------|-----------|
| Downloads from remote? | ✅ Yes | ✅ Yes |
| Updates working files? | ❌ No | ✅ Yes |
| Safe to run anytime? | ✅ Yes | ⚠️ May cause conflicts |
| Equivalent to | Just downloading | `git fetch` + `git merge` |

```bash
# Safe approach — inspect before merging
git fetch
git diff main origin/main   # see what's different
git merge origin/main       # merge when ready

# Quick approach
git pull                    # fetch + merge in one step
```

---

### Q39. What happens when you run `git clone <url>`? (List all 4 things)

**Answer:**
```bash
git clone https://github.com/hiteshchoudhary/golang.git
```

1. Creates a new folder `golang/` in your current directory
2. Downloads **ALL files, ALL commit history, ALL branches**
3. Automatically sets `origin` to point to the cloned URL
4. Creates a local `main` branch that tracks `origin/main`

```bash
cd golang
git remote -v
# origin  https://github.com/hiteshchoudhary/golang.git

git log --oneline
# (full history of that repo)

git branch
# * main
```

---

### Q40. You get "error: remote origin already exists" when running `git remote add origin`. How do you fix it?

**Answer:**
```bash
git remote add origin https://github.com/...
# error: remote origin already exists.
```

**Fix Option 1 — Change the URL:**
```bash
git remote set-url origin https://github.com/Shubh3009/correct-repo.git
```

**Fix Option 2 — Remove and re-add:**
```bash
git remote remove origin
git remote add origin https://github.com/Shubh3009/correct-repo.git
```

**Verify:**
```bash
git remote -v   # confirm new URL
```

---

## 🔐 SECTION 9 — SSH Authentication

---

### Q41. Why use SSH over HTTPS for GitHub authentication?

**Answer:**

| | HTTPS | SSH |
|--|-------|-----|
| Auth method | Username + Personal Access Token | Public/private key pair |
| Password needed each time? | Yes | No (after setup) |
| Multiple accounts | Hard | Easy via `~/.ssh/config` |

With HTTPS you get:
```
remote: Support for password authentication was removed.
fatal: Authentication failed.
```

With SSH (after setup), `git push` just works silently.

---

### Q42. You have two GitHub accounts. Explain how to configure SSH to handle both.

**Answer:**
Create separate key pairs and configure `~/.ssh/config`:

```bash
nano ~/.ssh/config
```
```
# Personal account
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal

# Work account
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
```

Then use the alias in remote URLs:
```bash
# Personal repo
git remote set-url origin git@github-personal:Shubh3009/my-repo.git

# Work repo
git remote set-url origin git@github-work:MyCompany/work-repo.git
```

Test each:
```bash
ssh -T git@github-personal   # Hi Shubh3009! You've successfully authenticated...
ssh -T git@github-work       # Hi WorkAccount! ...
```

---

### Q43. What does `cat ~/.ssh/id_ed25519_personal.pub` output, and what do you do with it?

**Answer:**
```bash
cat ~/.ssh/id_ed25519_personal.pub
```
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB... shubham@macbook
```

This is your **public key**. You paste it into GitHub:
> GitHub → Settings → SSH and GPG keys → New SSH key → paste → Save

**NEVER share** `~/.ssh/id_ed25519_personal` (no `.pub`) — that's the private key.

---

## ⚙️ SECTION 10 — Git Config

---

### Q44. What are the 3 levels of Git config? Which level wins?

**Answer:**

| Level | Flag | Location | Scope |
|-------|------|----------|-------|
| System | `--system` | `/Library/Developer/.../gitconfig` | All users on machine |
| Global | `--global` | `~/.gitconfig` | Current user, all repos |
| Local | (none) | `.git/config` | Current repo only |

**Priority: Local > Global > System** (most specific wins)

```bash
# Set globally (most common)
git config --global user.name "Shubham Kumar"
git config --global user.email "shubham@example.com"

# Set only for this repo (overrides global)
git config user.email "work@company.com"
```

---

### Q45. How do you see where each config value is coming from?

**Answer:**
```bash
git config --list --show-origin
```
```
file:/Library/.../gitconfig        core.autocrlf=input
file:/Users/shubhamkumar/.gitconfig  user.name=Shubham Kumar
file:/Users/shubhamkumar/.gitconfig  user.email=shubham@example.com
file:.git/config                     remote.origin.url=git@github-personal:...
```

Shows the **source file** for each setting — great for debugging wrong email/name in commits.

---

## 🔓 SECTION 11 — Open Source Workflow

---

### Q46. What is the complete open source contribution workflow? List all steps in order.

**Answer:**

```
1. Fork     → Copy the repo to your own GitHub account
2. Clone    → Download your fork locally
3. Branch   → Create a feature branch (NEVER work on main!)
4. Change   → Edit/create files
5. Commit   → Save your changes locally
6. Push     → Upload your branch to your fork on GitHub
7. PR       → Open a Pull Request to the original repo
8. Review   → Maintainer reviews and merges
```

**Your exact session (open-source-11):**
```bash
git clone https://github.com/Shubh3009/open-source-11.git
cd open-source-11
git checkout -b navbar            # Step 3
rm navbar1.html                   # Step 4
# create navbar.html              # Step 4
git add .                         # Step 5
git commit -m "add nav bar feature to code"  # Step 5
git remote set-url origin git@github-personal:Shubh3009/open-source-11.git
git push -u origin navbar         # Step 6
# Then open PR on GitHub          # Step 7
```

---

### Q47. You push your branch and get this error. What caused it and how do you fix it?

```
error: failed to push some refs
hint: Updates were rejected because the remote contains work you don't have.
```

**Answer:**
**Cause:** Someone else pushed to the same branch on GitHub after you last pulled. Your local branch is behind the remote.

**Fix:**
```bash
git pull --rebase   # fetch remote changes, replay your commits on top
git push            # now push succeeds
```

Or step by step:
```bash
git fetch
git rebase origin/main
git push
```

---

### Q48. Why should you NEVER commit directly to `main` in an open source project?

**Answer:**
1. **Main is the stable branch** — everyone using the project pulls from it. Broken code affects all users.
2. **PRs exist for review** — maintainers need to review and approve code before it merges.
3. **You may not have push access** to `main` anyway — GitHub protects it with branch rules.
4. **Isolation** — a feature branch keeps your work separate so you can make PRs, get feedback, and update without affecting others.

**Rule:**
```bash
# ❌ Wrong
git checkout main
# ... make changes ...
git commit -am "added navbar"
git push

# ✅ Correct
git checkout -b navbar-feature
# ... make changes ...
git commit -am "added navbar"
git push -u origin navbar-feature
# → Open Pull Request on GitHub
```

---

## 🧠 SECTION 12 — Mixed / Tricky Questions

---

### Q49. Sequence question — Put these in the correct order for a standard feature workflow:

```
A. git push -u origin feature
B. git checkout -b feature
C. git add .
D. git commit -m "added feature"
E. git checkout main
F. git merge feature
G. git branch -d feature
```

**Answer:** `E → B → C → D → A → (PR on GitHub or:) E → F → G`

For local-only workflow:
```bash
git checkout main          # E - ensure you're on main
git checkout -b feature    # B - create feature branch
# ... make changes ...
git add .                  # C - stage
git commit -m "added feature" # D - commit
git push -u origin feature # A - push to remote (for PR)

# After PR is merged OR for local merge:
git checkout main          # E
git merge feature          # F
git branch -d feature      # G
```

---

### Q50. What is the content of `.git/refs/heads/main` and how does it change after each commit?

**Answer:**
It contains the **SHA-1 hash of the latest commit on that branch** — just 40 characters + newline.

```bash
cat .git/refs/heads/main
# a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0

git commit -m "added footer"

cat .git/refs/heads/main
# f1e2d3c4b5a6f7e8d9c0b1a2f3e4d5c6b7a8f9e0   ← updated to new commit!
```

Every new commit updates this file to the new commit's hash. This is how Git tracks "the latest commit on each branch" without any database — just simple files!

---

### Q51. Spot the mistake and fix it:

```bash
git init
echo "<h1>Hello</h1>" > index.html
git commit -m "first commit"
```

**Answer:**
The mistake is **skipping `git add`**. You cannot commit without staging first.

```
# Error you'd see:
On branch main
No commits yet
Untracked files: index.html
nothing added to commit but untracked files present
```

**Fixed:**
```bash
git init
echo "<h1>Hello</h1>" > index.html
git add index.html              # ← MISSING STEP
git commit -m "first commit"   # now works!
```

---

### Q52. What is the output of `git status` after ALL conflicts are resolved but before `git commit`?

**Answer:**
```bash
git status
```
```
On branch main
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
    modified: index.html
```

Git knows conflicts are fixed (you ran `git add index.html`) but is waiting for you to finalize the merge with a commit:
```bash
git commit -m "merged footer branch"
```

---

> ← **[QA_Part1.md](./QA_Part1.md)** — Git Basics · Branches · Log & Diff · Merge

---

**Total: 52 Questions covering all 8 chapters**

| Section | Questions | Topics |
|---------|-----------|--------|
| 1 — Basics | Q1–Q8 | init, status, add, commit, -am, file states, HEAD |
| 2 — Branches | Q9–Q13 | branch internals, -b, checkout, file visibility, -d vs -D |
| 3 — Log & Diff | Q14–Q17 | log --oneline, diff, --staged, commit comparison |
| 4 — Merge | Q18–Q22 | fast-forward, 3-way, conflict markers, resolution |
| 5 — Stash | Q23–Q26 | stash use case, pop vs apply, cross-branch, stash list |
| 6 — Reset | Q27–Q30 | --soft/--mixed/--hard, HEAD~N, detached HEAD |
| 7 — Rebase | Q31–Q35 | rebase internals, conflict, abort, golden rule, vs merge |
| 8 — Remote | Q36–Q40 | remote add, -u flag, fetch vs pull, clone, errors |
| 9 — SSH | Q41–Q43 | SSH vs HTTPS, multi-account config, public key |
| 10 — Config | Q44–Q45 | 3 levels, priority, --show-origin |
| 11 — Open Source | Q46–Q48 | full workflow, push rejection, PR best practice |
| 12 — Mixed | Q49–Q52 | sequencing, .git internals, spot the bug, status output |
