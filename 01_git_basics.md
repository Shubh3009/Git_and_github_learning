# 📘 Chapter 1 — Git Basics: Init, Status, Add & Commit

> **Demo Project**: A simple website with `index.html`, `nav-bar.html`, `footer.html`  
> These are the EXACT files from your `gittwo/` session.

---

## 🚀 Setting Up a New Project

### Step 1 — Create your project folder and files

```bash
mkdir my-website
cd my-website
```

Now create your first file `index.html`:

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>git learning</title>
</head>
<body>
    Welcome to my website
</body>
</html>
```

---

## `git init` — Turn a folder into a Git repo

```bash
git init
```

**What happens internally:**
- Git creates a hidden `.git/` folder inside `my-website/`
- This folder stores ALL history, branches, configs
- Your files are NOT yet tracked — Git just knows about the folder

**Terminal output:**
```
Initialized empty Git repository in /my-website/.git/
```

**Before `git init`:**
```
my-website/
└── index.html          ← just a regular file, Git knows nothing
```

**After `git init`:**
```
my-website/
├── .git/               ← Git's brain lives here
│   ├── HEAD            ← "you are on branch: main"
│   ├── objects/        ← where commits/files are stored as blobs
│   └── refs/heads/     ← branch pointers (empty for now)
└── index.html
```

---

## `git status` — See the current state

```bash
git status
```

**Output right after `git init` (nothing staged yet):**
```
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        index.html

nothing added to commit but untracked files present
```

**What each line tells you:**
| Line | Meaning |
|------|---------|
| `On branch main` | You're on the main branch |
| `No commits yet` | No snapshots taken yet |
| `Untracked files: index.html` | Git sees it but isn't tracking it |

---

## `git add` — Move files to the Staging Area

Think of staging as a **draft box** before sending an email.

### Add a single file
```bash
git add index.html
```

**What happens:**
- Git takes a snapshot of `index.html` right now
- Stores it in `.git/objects/` as a blob
- Adds it to the **index** (staging area / draft box)
- File is now ready to be committed

### Add ALL files at once
```bash
git add .
```

**Run `git status` again after staging:**
```
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   index.html
```

> 💡 Notice the color change: **red** = not staged, **green** = staged and ready

---

## `git commit` — Save a permanent snapshot

```bash
git commit -m "add index.html"
```

**What happens:**
1. Git wraps the staged files into a **commit object**
2. The commit stores:  
   - A pointer to all staged files (as a tree)
   - Your name & email (from `git config`)
   - A timestamp
   - The commit message
   - A pointer to the **parent commit**
3. The branch pointer (`main`) now points to this new commit

**Terminal output:**
```
[main (root-commit) a1b2c3d] add index.html
 1 file changed, 10 insertions(+)
 create mode 100644 index.html
```

**What each part means:**
| Part | Meaning |
|------|---------|
| `main` | You committed to the main branch |
| `root-commit` | This is the very first commit (no parent) |
| `a1b2c3d` | Short SHA-1 hash — unique ID for this commit |
| `1 file changed` | How many files were in the snapshot |

---

## Full Demo Walkthrough — From scratch to 3 commits

```bash
# Step 1: Init
git init
# → Creates .git/

# Step 2: Create index.html (see content above)
# Step 3: Check status
git status
# → index.html is UNTRACKED (red)

# Step 4: Stage it
git add index.html
git status
# → index.html is STAGED (green)

# Step 5: Commit
git commit -m "add index.html"
# → First commit created!

# ---- Now add nav-bar.html ----
# Create nav-bar.html:
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
# → Second commit!

# ---- Now add footer.html ----
```

```html
<!-- footer.html -->
<footer>
  <p>&copy; 2024 My Company. All rights reserved.</p>
  <p>Follow us on social media.</p>
</footer>
```

```bash
git add footer.html
git commit -m "added footer"
# → Third commit!
```

**Commit history now looks like:**
```
a1b2c3d  add index.html          ← oldest
b2c3d4e  add navbar to code base
c3d4e5f  added footer            ← latest (HEAD → main)
```

---

## `git commit -am` — Shortcut: Add tracked files + Commit

```bash
git commit -am "updated main branch"
```

**This is equivalent to:**
```bash
git add index.html    # (only for already-tracked files)
git commit -m "updated main branch"
```

> ⚠️ **Does NOT work for brand new files.** If `hero-section.html` is new (never committed before), you must `git add hero-section.html` first.

---

## The 3 States of a File in Git

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  UNTRACKED         MODIFIED          STAGED       COMMITTED  │
│  (new file,   →  (you edited   →  (git add   →  (git commit │
│  Git ignores)     a tracked        ran)           ran)       │
│                   file)                                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Example with `index.html`:**
```bash
# State 1: UNTRACKED
echo "<h1>Hello</h1>" > index.html
git status   # → Untracked files: index.html

# State 2: STAGED
git add index.html
git status   # → Changes to be committed: index.html

# State 3: COMMITTED
git commit -m "add index"
git status   # → nothing to commit, working tree clean

# State 4: MODIFIED (after editing)
echo "<h2>New Section</h2>" >> index.html
git status   # → Changes not staged for commit: index.html
```

---

> **Next Chapter →** [02_branches.md](./02_branches.md) — Creating, switching, and deleting branches
