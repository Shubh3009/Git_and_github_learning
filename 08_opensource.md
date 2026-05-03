# 📘 Chapter 8 — Open Source Contribution Workflow

> **Real Session**: Your exact `open-source-11` contribution, step by step.

---

## The Open Source Workflow

```
[GitHub Repo]        [Your Fork]         [Your Local Machine]
     │                   │                       │
     │──── Fork ─────────►│                       │
     │                   │──── Clone ────────────►│
     │                   │                       │── Create branch
     │                   │                       │── Make changes
     │                   │                       │── Commit
     │                   │◄─── Push branch ───────│
     │◄─── Pull Request ──│                       │
     │──── Review & Merge─►│                       │
```

---

## Your Exact Session — Reconstructed

### Step 1: Clone the Repository

```bash
git clone https://github.com/Shubh3009/open-source-11.git
```

**Output:**
```
Cloning into 'open-source-11'...
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
Receiving objects: 100% (12/12), done.
```

```bash
ls -la
# open-source-11/    ← new folder
```

---

### Step 2: Enter the Repo and Inspect

```bash
cd open-source-11
ls
# index.html   navbar1.html   (existing files)

git status
# On branch main
# nothing to commit, working tree clean

git branch
# * main
```

---

### Step 3: Create a Feature Branch

> **Rule #1 of Open Source**: NEVER work directly on `main`.  
> Always create a feature branch so your changes are isolated.

```bash
git checkout -b navbar
git branch
#   main
# * navbar    ← you are here
```

---

### Step 4: Remove the Old Navbar File (cleanup)

```bash
rm navbar1.html
```

**What `rm` does:** This is a shell command (not Git) — it deletes the file from disk.

```bash
git status
# Changes not staged for commit:
#     deleted: navbar1.html
```

Git notices the file is gone but it's not staged yet.

---

### Step 5: Create a Better Navbar

Create `navbar.html` (the improved version):

```html
<!-- navbar.html -->
<nav class="navbar">
  <div class="brand">MyBrand</div>
  <ul class="nav-links">
    <li><a href="#home">Home</a></li>
    <li><a href="#about">About</a></li>
    <li><a href="#services">Services</a></li>
    <li><a href="#contact">Contact</a></li>
  </ul>
</nav>
```

```bash
git status
# Changes not staged for commit:
#     deleted:  navbar1.html
# Untracked files:
#     navbar.html
```

---

### Step 6: Stage and Commit All Changes

```bash
git add .
git status
# Changes to be committed:
#     deleted:  navbar1.html
#     new file: navbar.html

git commit -m "add nav bar feature to code"
```

---

### Step 7: Set Up SSH Remote (for your account)

```bash
git remote -v
# origin  https://github.com/Shubh3009/open-source-11.git (fetch)
# origin  https://github.com/Shubh3009/open-source-11.git (push)

# Switch to SSH (using your personal SSH alias)
git remote set-url origin git@github-personal:Shubh3009/open-source-11.git

git remote -v
# origin  git@github-personal:Shubh3009/open-source-11.git (fetch)
# origin  git@github-personal:Shubh3009/open-source-11.git (push)
```

---

### Step 8: Push the Feature Branch to GitHub

```bash
git push -u origin navbar
```

**Output:**
```
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 380 bytes | 380.00 KiB/s, done.
To github-personal:Shubh3009/open-source-11.git
 * [new branch]      navbar -> navbar
Branch 'navbar' set up to track remote branch 'navbar' from 'origin'.
```

**What happened:**
- A new `navbar` branch was created on GitHub
- Your commits were uploaded
- GitHub will now show a banner: **"Compare & pull request"**

---

### Step 9: Open a Pull Request on GitHub

1. Go to `https://github.com/Shubh3009/open-source-11`
2. You'll see: **"navbar had recent pushes — Compare & pull request"**
3. Click it
4. Fill in the PR form:

```
Title: Add improved navbar component

Description:
- Removed old navbar1.html (had broken links)
- Added new navbar.html with proper class names
- Links to all main sections: Home, About, Services, Contact
```

5. Click **"Create Pull Request"**

---

## Complete Flow Diagram

```
local machine                         GitHub
──────────────                        ──────

git clone open-source-11  ←──────── github.com/Shubh3009/open-source-11
         │
         ▼
git checkout -b navbar
         │
         ▼
rm navbar1.html
create navbar.html
         │
         ▼
git add .
git commit -m "add nav bar feature"
         │
         ▼
git push -u origin navbar  ──────── github.com: new branch 'navbar'
                                              │
                                              ▼
                                    "Compare & pull request" ──►
                                    Open PR → Review → Merged!
```

---

## Common Issues & Fixes

### Issue 1: Push rejected — wrong credentials

```bash
git push -u origin main
# remote: Support for password authentication was removed
# fatal: Authentication failed
```

**Fix:** Switch to SSH:
```bash
git remote set-url origin git@github-personal:Shubh3009/Rough_github.git
git push -u origin main
```

---

### Issue 2: Remote already exists

```bash
git remote add origin https://github.com/...
# error: remote origin already exists.
```

**Fix:**
```bash
git remote set-url origin https://github.com/new-url.git
# OR
git remote remove origin
git remote add origin https://github.com/new-url.git
```

---

### Issue 3: Branch behind remote

```bash
git push
# error: failed to push some refs
# hint: Updates were rejected because the remote contains work you don't have
```

**Fix:**
```bash
git pull --rebase   # pull remote changes first, then push
git push
```

---

## Summary — Key Commands for Open Source

```bash
# Clone a repo
git clone https://github.com/user/repo.git

# Always work on a feature branch
git checkout -b my-feature

# Make changes, then:
git add .
git commit -m "what I changed"

# Push your branch
git push -u origin my-feature

# Then go to GitHub and open a Pull Request!
```

---

> 📚 **All Chapters:**
> - [01 — Git Basics](./01_git_basics.md)
> - [02 — Branches](./02_branches.md)
> - [03 — Log & Diff](./03_log_and_diff.md)
> - [04 — Merge & Conflicts](./04_merge.md)
> - [05 — Stash & Reset](./05_stash_reset.md)
> - [06 — Rebase](./06_rebase.md)
> - [07 — Remote & GitHub](./07_remote_github.md)
> - [08 — Open Source Workflow](./08_opensource.md) ← You are here
