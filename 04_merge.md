# 📘 Chapter 4 — Git Merge & Conflict Resolution

> **Scenario**: You built the navbar on `nav-bar` branch and footer on `footer` branch.  
> Now you need to bring all changes back into `main`.

---

## What is Merging?

Merging takes the work from one branch and **integrates it into another**.

```
Before merge:
  main    ──► C1 ──► C2
                      ↑
  nav-bar ──► C1 ──► C2 ──► C3 (nav-bar.html added)

After: git merge nav-bar (while on main)
  main    ──► C1 ──► C2 ──► C3   ← main now has nav-bar.html too!
```

---

## Type 1: Fast-Forward Merge

Happens when `main` has NO new commits since `nav-bar` branched off.  
Git simply moves the `main` pointer forward — no merge commit needed.

### Demo

```bash
# Starting state: only index.html committed on main
git log --oneline
# e4b9d8g  add index.html   ← main is here

# Create and switch to nav-bar branch
git checkout -b nav-bar

# Create nav-bar.html
```

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

# Now merge into main
git checkout main
git merge nav-bar -m "merging nav-bar"
```

**Output:**
```
Updating e4b9d8g..b7e2a1d
Fast-forward
 nav-bar.html | 10 ++++++++++
 1 file changed, 10 insertions(+)
 create mode 100644 nav-bar.html
```

**What happened:**
```
Before:  main ──► C1
                   \
         nav-bar ──► C2 (nav-bar.html)

After:   main ──► C1 ──► C2   (main pointer just moved forward)
```

```bash
ls
# index.html  nav-bar.html   ← nav-bar.html is now on main!

git log --oneline
# b7e2a1d  add navbar to code base
# e4b9d8g  add index.html
```

Delete the branch (it's merged):
```bash
git branch -d nav-bar
# Deleted branch nav-bar (was b7e2a1d).
```

---

## Type 2: Three-Way Merge (Merge Commit)

Happens when BOTH branches have new commits after they diverged.

### Demo

```bash
# On main: add hero-section.html
git add hero-section.html
git commit -m "add hero section of code base"

# Meanwhile, on footer branch: add footer.html
git checkout -b footer
git add footer.html
git commit -m "added footer"
```

**Now both branches have diverged:**
```
main   ──► C1 ──► C2 (hero-section)
                   \
footer              C3 (footer.html)
```

```bash
# Go back to main and merge footer
git checkout main
git merge footer -m "merging footer"
```

**Output:**
```
Merge made by the 'ort' strategy.
 footer.html | 4 ++++
 1 file changed, 4 insertions(+)
 create mode 100644 footer.html
```

Git creates a **merge commit** (C4) that has TWO parents:
```
main   ──► C1 ──► C2 ──────────────► C4 (merge commit)
                    \               /
footer               C3 (footer)──►
```

```bash
git log --oneline
# 4572c76  merging footer       ← this is the merge commit (C4)
# a8f3b2c  added footer
# c6d1f0e  add hero section
# e4b9d8g  add index.html
```

---

## Type 3: Merge Conflict ⚠️

Happens when **both branches edited the same part of the same file**.

### Demo — Conflict in `index.html`

**On `main`**, `index.html` has:
```html
<body>
    i love to add nav bar in my project
    footer was added successfully
</body>
```

**On `footer` branch**, someone edited `index.html` to:
```html
<body>
    i love to add nav bar in my project
    footer was added successfully
    update index file with footer code     ← different line added
</body>
```

**On `main` branch**, you also edited `index.html` to:
```html
<body>
    i love to add nav bar in my project
    footer was added successfully
    added footer in index.html             ← different line added here too
</body>
```

```bash
git checkout main
git merge footer
```

**Output:**
```
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

**Git pauses the merge. `index.html` now contains conflict markers:**
```html
<body>
    i love to add nav bar in my project
    footer was added successfully
<<<<<<< HEAD
    added footer in index.html
=======
    update index file with footer code
>>>>>>> footer
</body>
```

**Reading conflict markers:**
```
<<<<<<< HEAD               ← start of YOUR changes (current branch = main)
    added footer in index.html
=======                    ← divider
    update index file with footer code
>>>>>>> footer             ← end of INCOMING changes (from footer branch)
```

### Resolving the Conflict

Edit the file manually — keep what you want:

```html
<body>
    i love to add nav bar in my project
    footer was added successfully
    added footer in index.html
    update index file with footer code
</body>
```

Then tell Git you resolved it:
```bash
git add index.html        # mark as resolved
git status
# All conflicts fixed but you are still merging.
# Changes to be committed: index.html

git commit -m "merged footer branch"
# → Merge commit created with both changes!
```

---

## Merge vs No-Merge Summary

| Situation | Merge Type | Creates New Commit? |
|-----------|-----------|-------------------|
| `main` not changed since branch | Fast-forward | ❌ No |
| Both branches have new commits | 3-way merge | ✅ Yes (merge commit) |
| Same lines edited in both | Conflict | ✅ Yes (after manual fix) |

---

## Full Log After All Merges (from your session)

```bash
git log --oneline
```
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

---

> **Next Chapter →** [05_stash_reset.md](./05_stash_reset.md) — Stash and Reset
