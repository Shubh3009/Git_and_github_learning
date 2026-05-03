# 📘 Chapter 3 — Git Log & Diff

> **Scenario**: You've made several commits across `main`, `nav-bar`, and `footer` branches.  
> Now you want to inspect what changed, when, and compare versions.

---

## `git log` — Full Commit History

```bash
git log
```

**Output:**
```
commit 5628807f3a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5   ← full SHA hash
Author: Shubham Kumar <shubham@example.com>
Date:   Sun May 3 17:30:00 2026 +0530

    changed index and footer

commit 4572c76a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7
Author: Shubham Kumar <shubham@example.com>
Date:   Sun May 3 16:00:00 2026 +0530

    merged footer branch

commit 4dd2a6db3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8
Author: Shubham Kumar <shubham@example.com>
Date:   Sun May 3 14:00:00 2026 +0530

    adding readme.md
```

**What each field means:**
| Field | Meaning |
|-------|---------|
| `commit 5628807...` | Full SHA-1 hash — unique fingerprint of this snapshot |
| `Author` | Who made the commit (from `git config`) |
| `Date` | When the commit was made |
| Message | What you wrote in `-m "..."` |

---

## `git log --oneline` — Compact View

```bash
git log --oneline
```

**Output (from your actual session):**
```
5628807  changed index and footer
4572c76  merged footer branch
4dd2a6d  adding readme.md
a8f3b2c  added footer in index.html
b7e2a1d  merging nav-bar
c6d1f0e  add hero section of code base
d5c0e9f  add navbar to code base
e4b9d8g  add index.html
```

> 💡 **Reading the log**: Newest commits are at the TOP. The bottom is the oldest (first) commit.

---

## `git diff` — What Changed?

### Case 1: Working Directory vs Staging Area

You edit `index.html` (add a new line) but haven't staged it yet:

```bash
# Current index.html has:
# <body>
#   i love to add nav bar in my project
# </body>

# You add a new line manually:
# <body>
#   i love to add nav bar in my project
#   footer was added successfully        ← NEW LINE
# </body>

git diff
```

**Output:**
```diff
diff --git a/index.html b/index.html
index a1b2c3d..e4f5g6h 100644
--- a/index.html
+++ b/index.html
@@ -8,6 +8,7 @@
 <body>
     i love to add nav bar in my project
+    footer was added successfully
 </body>
```

**How to read this:**
| Symbol | Meaning |
|--------|---------|
| `---` | The OLD version (before your edit) |
| `+++` | The NEW version (after your edit) |
| `-` (red) | Line that was removed |
| `+` (green) | Line that was added |
| No symbol | Unchanged context line |
| `@@ -8,6 +8,7 @@` | "In old file, 6 lines starting at line 8; in new file, 7 lines starting at line 8" |

---

### Case 2: Staged Changes vs Last Commit

After you run `git add index.html`:

```bash
git diff            # shows NOTHING (no unstaged changes)
git diff --staged   # shows the staged change vs last commit
```

**Output of `git diff --staged`:**
```diff
diff --git a/index.html b/index.html
--- a/index.html
+++ b/index.html
@@ -8,5 +8,6 @@
 <body>
     i love to add nav bar in my project
+    footer was added successfully
 </body>
```

---

### Case 3: Compare Two Commits

From your session — comparing two specific commits:

```bash
git diff 5628807 4dd2a6d
```

**What this does:** Shows everything that changed between commit `4dd2a6d` and commit `5628807`.

**Example output:**
```diff
diff --git a/footer.html b/footer.html
--- a/footer.html
+++ b/footer.html
@@ -1,3 +1,4 @@
 <footer>
   <p>&copy; 2024 My Company. All rights reserved.</p>
+  <p>Follow us on social media.</p>
 </footer>

diff --git a/index.html b/index.html
--- a/index.html
+++ b/index.html
@@ -8,5 +8,7 @@
 <body>
     i love to add nav bar in my project
+    footer was added successfully
+    images are being added to the project
 </body>
```

> 💡 **Order matters!** `git diff A B` = "what was added/removed going FROM A TO B"
> - `git diff 4dd2a6d 5628807` = what was added after `4dd2a6d`
> - `git diff 5628807 4dd2a6d` = reverses it (shows those lines as removed)

---

## Full Demo — Tracking Changes to `footer.html`

**Start state — `footer.html` has:**
```html
<footer>
  <p>&copy; 2024 My Company. All rights reserved.</p>
</footer>
```

```bash
git add footer.html
git commit -m "initial footer"
```

**You edit `footer.html` to add social media line:**
```html
<footer>
  <p>&copy; 2024 My Company. All rights reserved.</p>
  <p>Follow us on social media.</p>   ← ADDED
</footer>
```

```bash
git diff
# Shows: + <p>Follow us on social media.</p>

git add footer.html
git diff            # → nothing (it's staged now)
git diff --staged   # → shows the staged addition

git commit -m "changed index and footer"

git log --oneline
# 5628807 changed index and footer    ← new commit at top
# 4dd2a6d initial footer
```

Now compare them:
```bash
git diff 4dd2a6d 5628807
# → shows the social media line was added
```

---

## `cat <file>` — Read a file directly

From your session:
```bash
cat footer.html
```
**Output:**
```html
<footer>
  <p>&copy; 2023 Your Company. All rights reserved.</p>
  <p>Follow us on social media:</p>
</footer>
```

> Used to quickly check current file content without opening an editor.

---

## Summary Table

| Command | What it compares |
|---------|-----------------|
| `git diff` | Working dir vs staging area (unstaged changes) |
| `git diff --staged` | Staging area vs last commit (staged changes) |
| `git diff A B` | Commit A vs Commit B |
| `git log` | Full history with all metadata |
| `git log --oneline` | Compact one-line history |
| `cat file.html` | Show raw file content |

---

> **Next Chapter →** [04_merge.md](./04_merge.md) — Merging branches and resolving conflicts
