# 📘 Chapter 2 — Git Branches

> **Scenario**: You're building a website. The `main` branch has the stable code.  
> You need to add a navbar — but don't want to break `main` while working on it.

---

## What is a Branch?

A **branch** is just a lightweight pointer to a commit. Creating one never copies files.

**Visual:**
```
main ──► [commit-1: index.html] ──► [commit-2: index + hero] ──► HEAD
```
After `git branch nav-bar`:
```
main    ──► [C1] ──► [C2] ──► HEAD (still on main)
nav-bar ──► [C2]              (points to same commit as main!)
```
> Both branches point to the same commit — no files were duplicated.

---

## `git branch` — List all branches

```bash
git branch
```

**Output:**
```
* main
```
The `*` means you are currently ON this branch.

After creating more branches:
```bash
git branch
```
```
* main
  nav-bar
  footer
  bugfix
```

---

## `git branch <name>` — Create a new branch

```bash
git branch nav-bar
```

**What happens:**
- A new file is created at `.git/refs/heads/nav-bar`
- That file contains the SHA hash of the current commit (same as `main`)
- You stay on `main` — this command does NOT switch

**Verify:**
```bash
ls .git/refs/heads/
# Output: main  nav-bar
```

```bash
cat .git/refs/heads/nav-bar
# Output: a1b2c3d4e5f6...  (same hash as main right now)
```

---

## `git checkout <branch>` — Switch to a branch

```bash
git checkout nav-bar
```

**What Git does step by step:**
1. Reads `.git/refs/heads/nav-bar` to find the target commit
2. Updates `.git/HEAD` to say `ref: refs/heads/nav-bar`
3. Loads that commit's file tree into your working directory
4. Your terminal now shows `(nav-bar)` branch

**Before checkout:**
```bash
cat .git/HEAD
# ref: refs/heads/main
```

**After `git checkout nav-bar`:**
```bash
cat .git/HEAD
# ref: refs/heads/nav-bar
```

---

## `git checkout -b <name>` — Create AND switch in one step

```bash
git checkout -b footer
```

**Equivalent to:**
```bash
git branch footer       # create
git checkout footer     # switch
```

**Output:**
```
Switched to a new branch 'footer'
```

---

## Full Demo — Navbar Feature Branch Workflow

```bash
# You're on main with index.html committed

git branch nav-bar         # create branch
git checkout nav-bar       # switch to it
git branch                 # confirm
```
```
  main
* nav-bar                  ← asterisk = you are here
```

Now create `nav-bar.html`:
```html
<!-- nav-bar.html -->
<nav>
  <ul>
    <li><a href="#home">Home</a></li>
    <li><a href="#about">About</a></li>
    <li><a href="#services">Services</a></li>
    <li><a href="#contact">Contact</a></li>
  </ul>
</nav>
```

```bash
git add nav-bar.html
git commit -m "add navbar to code base"
```

**History on nav-bar branch:**
```
C1 (index.html) ──► C2 (nav-bar.html)   ← HEAD → nav-bar
                ↑
               main (still points to C1!)
```

Now switch back to main:
```bash
git checkout main
ls
# Output: index.html         ← nav-bar.html is GONE from view!
```

> 💡 `nav-bar.html` is NOT deleted — it lives in the `nav-bar` branch. When you switch back to `nav-bar`, it reappears.

```bash
git checkout nav-bar
ls
# Output: index.html  nav-bar.html   ← it's back!
```

---

## `git branch -d <name>` — Delete a branch (safe)

```bash
git branch -d nav-bar
```

**What "safe" means:**
- Git checks if this branch's commits are merged into another branch
- If NOT merged → refuses to delete (protects your work)
- If merged → deletes the branch pointer only (commits still exist in history)

**Output after deleting a merged branch:**
```
Deleted branch nav-bar (was b2c3d4e).
```

**Force delete (even if not merged):**
```bash
git branch -D nav-bar
```
> ⚠️ Use `-D` only if you're sure you don't need those commits.

---

## Branch State Diagram — Your Full Session

```
                         C1
main    ──────────────── index.html
        \
nav-bar  ── C2 (nav-bar.html)     [merged → deleted]
        \
footer   ── C3 (footer.html)     [merged → kept]
        \
bugfix   ── C4 (fixes)            [rebased onto main]
```

---

## 💡 Key Insight: Branches are Cheap!

A branch is literally just a **41-byte file** (a SHA-1 hash + newline):
```bash
cat .git/refs/heads/main
# 3a8f2b1c9d4e6f7a8b9c0d1e2f3a4b5c6d7e8f9a
```
That's it. No copying. No extra disk space. Create as many as you want!

---

> **Next Chapter →** [03_log_and_diff.md](./03_log_and_diff.md) — Viewing history and comparing changes
