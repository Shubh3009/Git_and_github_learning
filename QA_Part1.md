# 🧪 Git & GitHub — Practice Questions (Part 1 of 2)
> Covers: Git Basics · Branches · Log & Diff · Merge & Conflicts

---

## 📦 SECTION 1 — Git Basics (init, status, add, commit)

---

### Q1. What does `git init` do? What is created inside the project folder?

**Answer:**
`git init` turns a regular folder into a Git repository by creating a hidden `.git/` directory.

```bash
mkdir my-website && cd my-website
git init
# Output: Initialized empty Git repository in /my-website/.git/
```

The `.git/` folder contains:
| File/Folder | Purpose |
|-------------|---------|
| `HEAD` | Points to the current branch |
| `objects/` | Stores all commits, files (blobs), trees |
| `refs/heads/` | Branch pointers (each branch = one file) |
| `config` | Local repo configuration |

> Your project files are NOT tracked yet — `git init` only creates the tracking infrastructure.

---

### Q2. You just created `index.html` in a new repo. What does `git status` show, and why?

**Answer:**

```bash
git status
```
```
On branch main
No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        index.html

nothing added to commit but untracked files present
```

`index.html` is **untracked** because Git has never been told to watch this file. `git init` doesn't auto-track files — you must explicitly `git add` them.

---

### Q3. What is the Staging Area? Why does Git have it?

**Answer:**
The **Staging Area (Index)** is a buffer between your working directory and the repository.

```
Working Directory  →(git add)→  Staging Area  →(git commit)→  Repository
```

**Why it exists:** It lets you choose *exactly which changes* go into a commit. For example, you edited 3 files but only want to commit 1:

```bash
git add nav-bar.html      # only stage this one
git commit -m "add navbar"  # only nav-bar.html goes in
# index.html and footer.html are NOT committed yet
```

---

### Q4. What is the difference between `git add index.html` and `git add .`?

**Answer:**

| Command | What it stages |
|---------|---------------|
| `git add index.html` | Only `index.html` |
| `git add .` | ALL changed/new files in the current directory and subdirectories |

**Demo:**
```bash
# You have 3 new files: index.html, nav-bar.html, footer.html

git add index.html        # stages only index.html
git status
# Changes to be committed: index.html
# Untracked files: nav-bar.html, footer.html

git add .                 # stages everything remaining
git status
# Changes to be committed: index.html, nav-bar.html, footer.html
```

---

### Q5. What does a commit object store internally?

**Answer:**
A commit stores 5 things:

```bash
git commit -m "add index.html"
# [main (root-commit) a1b2c3d] add index.html
```

1. **Tree** — pointer to the snapshot of all staged files
2. **Author** — name + email (from `git config`)
3. **Timestamp** — date and time
4. **Message** — what you wrote in `-m "..."`
5. **Parent** — SHA hash of the previous commit (none for the very first commit → called `root-commit`)

---

### Q6. What does `git commit -am "message"` do? When does it fail?

**Answer:**
`-a` = auto-stage all **modified tracked** files, `-m` = commit message.

```bash
git commit -am "updated navbar"
```

**Equivalent to:**
```bash
git add index.html     # (auto-stages tracked modified files)
git commit -m "updated navbar"
```

**When it FAILS:**
```bash
touch hero-section.html   # brand new file, never committed
git commit -am "added hero"
# hero-section.html is NOT included — it's untracked!
```
Fix:
```bash
git add hero-section.html   # must add new files explicitly
git commit -m "added hero"
```

---

### Q7. Describe all 4 states a file can be in within Git.

**Answer:**

```
UNTRACKED → STAGED → COMMITTED → MODIFIED
```

| State | What it means | How to get there |
|-------|--------------|-----------------|
| **Untracked** | New file, Git doesn't know it | Create a new file |
| **Staged** | Marked to go into next commit | `git add <file>` |
| **Committed** | Saved permanently in history | `git commit` |
| **Modified** | Tracked file changed after last commit | Edit a committed file |

**Demo with `index.html`:**
```bash
echo "<h1>Hello</h1>" > index.html   # UNTRACKED
git add index.html                   # STAGED
git commit -m "add index"            # COMMITTED
echo "<p>More</p>" >> index.html     # MODIFIED
```

---

### Q8. What does `cat .git/HEAD` show? How does it change?

**Answer:**
`HEAD` is a file that tells Git "which branch are you currently on."

```bash
cat .git/HEAD
# ref: refs/heads/main    ← you're on main
```

After switching branches:
```bash
git checkout nav-bar
cat .git/HEAD
# ref: refs/heads/nav-bar  ← now on nav-bar
```

After checking out a specific commit (detached HEAD):
```bash
git checkout 4dd2a6d
cat .git/HEAD
# 4dd2a6d3b4c5d6...    ← raw hash, not a branch name!
```

---

## 🌿 SECTION 2 — Git Branches

---

### Q9. What is a Git branch internally? Why are branches called "cheap"?

**Answer:**
A branch is literally a **file** inside `.git/refs/heads/` containing a 40-character SHA-1 hash (the latest commit):

```bash
cat .git/refs/heads/main
# 3a8f2b1c9d4e6f7a8b9c0d1e2f3a4b5c6d7e8f9a
```

That's it — 41 bytes. No file copying. No extra disk space. This is why you can create hundreds of branches instantly.

```bash
git branch nav-bar
ls .git/refs/heads/
# main  nav-bar   ← nav-bar is just a new 41-byte file
```

---

### Q10. What is the difference between `git branch nav-bar` and `git checkout -b nav-bar`?

**Answer:**

| Command | Creates branch? | Switches to it? |
|---------|----------------|----------------|
| `git branch nav-bar` | ✅ Yes | ❌ No (you stay on current) |
| `git checkout -b nav-bar` | ✅ Yes | ✅ Yes |

```bash
# Using git branch (two steps needed)
git branch nav-bar       # create
git checkout nav-bar     # switch

# Shortcut (one step)
git checkout -b nav-bar  # create + switch
```

---

### Q11. You're on `main`. You `git checkout nav-bar`. What 3 things happen internally?

**Answer:**
1. Git reads `.git/refs/heads/nav-bar` to find the target commit hash
2. Updates `.git/HEAD` from `ref: refs/heads/main` → `ref: refs/heads/nav-bar`
3. Updates your **working directory files** to match the state of `nav-bar`'s commit

```bash
cat .git/HEAD         # ref: refs/heads/main
git checkout nav-bar
cat .git/HEAD         # ref: refs/heads/nav-bar
ls                    # files change to match nav-bar's snapshot!
```

---

### Q12. You're on `nav-bar` and you run `git checkout main`. `nav-bar.html` disappears from `ls`. Is it deleted? Explain.

**Answer:**
**No, it is NOT deleted.** `nav-bar.html` exists in the `nav-bar` branch's commit history — it's stored in `.git/objects/`. When you switch to `main`, Git restores `main`'s file snapshot, which doesn't include `nav-bar.html`.

```bash
git checkout main
ls
# index.html    ← nav-bar.html is gone from VIEW, not from Git!

git checkout nav-bar
ls
# index.html  nav-bar.html  ← it's back!
```

---

### Q13. What is the difference between `git branch -d` and `git branch -D`?

**Answer:**

| Flag | Behaviour |
|------|----------|
| `-d` | **Safe delete** — refuses if branch has unmerged commits |
| `-D` | **Force delete** — deletes regardless, even with unmerged work |

```bash
# Safe delete after merging
git merge nav-bar
git branch -d nav-bar    # works fine
# Deleted branch nav-bar (was b2c3d4e).

# Force delete without merging
git branch -D nav-bar    # deletes even if not merged ⚠️
```

> Deleting a branch only removes the pointer. The commits still exist in `.git/objects/` until garbage collected.

---

## 🔍 SECTION 3 — Log & Diff

---

### Q14. What is the difference between `git log` and `git log --oneline`?

**Answer:**

`git log` — full details per commit:
```
commit 5628807f3a1b2c3d...
Author: Shubham Kumar <shubham@example.com>
Date:   Sun May 3 17:30 2026

    changed index and footer
```

`git log --oneline` — compact one-line view:
```
5628807  changed index and footer
4572c76  merged footer branch
4dd2a6d  adding readme.md
```

> Newest commit is always at the **TOP**. Bottom = oldest.

---

### Q15. You edited `footer.html` but haven't staged it. Which diff command shows the change?

**Answer:**
```bash
git diff
```
This compares **working directory** vs **staging area** (last `git add`).

```diff
diff --git a/footer.html b/footer.html
--- a/footer.html
+++ b/footer.html
@@ -1,3 +1,4 @@
 <footer>
   <p>&copy; 2024 My Company.</p>
+  <p>Follow us on social media.</p>
 </footer>
```

`+` = added line (green), `-` = removed line (red), no symbol = unchanged.

---

### Q16. After `git add footer.html`, what does `git diff` show? What shows the change now?

**Answer:**
- `git diff` → **shows nothing** (no unstaged changes remain)
- `git diff --staged` → shows what is staged vs the last commit

```bash
git add footer.html
git diff            # (blank output)
git diff --staged   # shows the staged addition
```

| Command | Compares |
|---------|---------|
| `git diff` | Working dir ↔ Staging area |
| `git diff --staged` | Staging area ↔ Last commit |

---

### Q17. What does `git diff 5628807 4dd2a6d` do? Does order matter?

**Answer:**
It compares two specific commits by their hash — showing what changed going **from** `5628807` **to** `4dd2a6d`.

**Yes, order matters!**
```bash
git diff A B    # shows additions/removals going from A → B
git diff B A    # reverses: what was "added" in A→B now shows as "removed"
```

Example:
```bash
git diff 4dd2a6d 5628807   # shows lines ADDED after 4dd2a6d
git diff 5628807 4dd2a6d   # reverses — those same lines show as REMOVED
```

---

## 🔀 SECTION 4 — Merge & Conflict Resolution

---

### Q18. What is a fast-forward merge? When does it happen?

**Answer:**
A fast-forward merge happens when the **target branch (main) has NO new commits** since the feature branch was created. Git simply moves the `main` pointer forward — no new commit is created.

```
Before:  main ──► C1
                    \
         nav-bar ──► C2 (nav-bar.html)

After:   main ──► C1 ──► C2   ← main pointer moved forward
```

```bash
git checkout main
git merge nav-bar
# Output: Fast-forward
#  nav-bar.html | 10 ++++++++++
```

---

### Q19. What is a three-way merge? What extra thing does Git create?

**Answer:**
A three-way merge happens when **both branches have new commits** after diverging. Git creates a **merge commit** that has TWO parent commits.

```
main:    C1 ──► C2 (hero) ──────────────► C4 (merge commit ← 2 parents)
                  \                      /
footer:            C3 (footer) ─────────
```

```bash
git checkout main
git merge footer -m "merging footer"
# Output: Merge made by the 'ort' strategy.
```

```bash
git log --oneline
# 4572c76  merging footer    ← this is C4, the merge commit
# a8f3b2c  added footer
# c6d1f0e  add hero section
```

---

### Q20. A merge conflict happened in `index.html`. Show what the file looks like and the exact steps to resolve it.

**Answer:**
Git inserts **conflict markers** into the file:

```html
<body>
    footer was added successfully
<<<<<<< HEAD
    added footer in index.html       ← YOUR change (current branch)
=======
    update index file with footer code  ← INCOMING change (other branch)
>>>>>>> footer
</body>
```

**Resolution steps:**
```bash
# Step 1: Manually edit — remove markers, keep what you want
# <body>
#     footer was added successfully
#     added footer in index.html
#     update index file with footer code
# </body>

# Step 2: Mark as resolved
git add index.html

# Step 3: Check status
git status
# All conflicts fixed but you are still merging.

# Step 4: Complete the merge
git commit -m "merged footer branch"
```

---

### Q21. Fill in the table:

| Situation | Merge Type | Creates New Commit? |
|-----------|-----------|-------------------|
| `main` has no new commits since branching | ? | ? |
| Both branches diverged with new commits | ? | ? |
| Same lines edited on both branches | ? | ? |

**Answer:**

| Situation | Merge Type | Creates New Commit? |
|-----------|-----------|-------------------|
| `main` has no new commits since branching | Fast-forward | ❌ No |
| Both branches diverged with new commits | 3-way merge | ✅ Yes |
| Same lines edited on both branches | Conflict merge | ✅ Yes (after manual fix) |

---

### Q22. After a fast-forward merge of `nav-bar` into `main`, should you delete the `nav-bar` branch? How?

**Answer:**
Yes — after merging, the branch pointer is no longer needed. The commits still exist in `main`'s history.

```bash
git branch -d nav-bar
# Deleted branch nav-bar (was b7e2a1d).
```

Use `-d` (safe delete) — Git will verify it's merged before deleting.

---

> **→ Continue in [QA_Part2.md](./QA_Part2.md)** — Stash · Reset · Rebase · GitHub · SSH · Open Source
