# 📘 Chapter 5 — Git Stash & Reset

---

## PART A: Git Stash

### What is Stash?

`git stash` is a **temporary clipboard** for your uncommitted work.  
It saves your changes and reverts your working directory to the last clean commit.

**Use case:** You're halfway through editing `footer.html` when your manager says  
"Drop everything — fix a bug on `bugfix` branch NOW!"  
You can't commit half-done work. Stash saves it!

---

### `git stash` — Save your work temporarily

#### Setup: You're on `footer` branch, editing `index.html`

Current `index.html` on footer branch:
```html
<body>
    i love to add nav bar in my project
    footer was added successfully
</body>
```

You add a new line (uncommitted):
```html
<body>
    i love to add nav bar in my project
    footer was added successfully
    update index file with footer code    ← WIP, not committed
</body>
```

```bash
git status
# Changes not staged for commit:
#     modified: index.html

# Emergency! Need to switch branches
git stash
```

**Output:**
```
Saved working directory and index state WIP on footer: 4dd2a6d adding readme.md
```

**What happened:**
1. Git saves your `index.html` changes to a stash stack
2. `index.html` is **reverted** to its last committed state
3. Working directory is now clean

```bash
git status
# nothing to commit, working tree clean   ← clean!

cat index.html
# <body>
#     i love to add nav bar in my project
#     footer was added successfully       ← WIP line is GONE (temporarily)
# </body>
```

---

### `git stash pop` — Restore your saved work

```bash
# After dealing with the emergency:
git checkout footer      # back to footer branch
git stash pop
```

**Output:**
```
On branch footer
Changes not staged for commit:
    modified: index.html

Dropped refs/stash@{0}
```

```bash
cat index.html
# <body>
#     i love to add nav bar in my project
#     footer was added successfully
#     update index file with footer code    ← WIP line is BACK!
# </body>
```

> 💡 `stash pop` = apply stash + remove it from the stack  
> `stash apply` = apply stash + KEEP it in the stack (for reuse)

---

### Stash Across Branches — From Your Session

```bash
# On footer branch with uncommitted changes
git stash

# Switch to bugfix branch
git checkout bugfix

# Restore stash ON bugfix (stash works across branches!)
git stash pop

# Your footer edits are now visible on bugfix branch
```

---

### `git stash list` — See all stashes

```bash
git stash list
```

**Output (if you stashed multiple times):**
```
stash@{0}: WIP on footer: 4dd2a6d adding readme.md      ← most recent
stash@{1}: WIP on main:   5628807 changed index and footer
stash@{2}: WIP on bugfix: a1b2c3d initial
```

Think of it as a **stack** — newest on top.

---

### Stash Command Summary

| Command | What it does |
|---------|-------------|
| `git stash` | Save changes, clean working dir |
| `git stash pop` | Restore last stash + delete it from stack |
| `git stash apply` | Restore last stash, keep it in stack |
| `git stash list` | List all saved stashes |
| `git stash drop` | Delete the latest stash |
| `git stash clear` | Delete ALL stashes |

---

---

## PART B: Git Reset

### What is Reset?

`git reset` **moves the branch pointer backward** to a previous commit. It "undoes" commits.

---

### The 3 Types of Reset

Think of it as: "How far back do you want to go, and what do you keep?"

#### Setup — You're on main with this history:
```
C1 (add index)
C2 (add navbar)
C3 (add footer)      ← HEAD (current)
```

---

### `git reset --soft HEAD~1` — Undo commit, keep changes staged

```bash
git reset --soft HEAD~1
```

**What changes:**
- `HEAD` moves back to `C2`
- `C3`'s changes remain **staged** (in the staging area)
- Your files are unchanged

```bash
git status
# Changes to be committed:
#     modified: footer.html    ← still staged, just uncommitted
```

> Use this when: You want to **rewrite the commit message** or **add more files** before committing.

---

### `git reset HEAD~1` (--mixed, the DEFAULT) — Undo commit and unstage

```bash
git reset HEAD~1
# OR
git reset --mixed HEAD~1
```

**What changes:**
- `HEAD` moves back to `C2`
- `C3`'s changes are **unstaged** (back to "modified" state)
- Your files are unchanged

```bash
git status
# Changes not staged for commit:
#     modified: footer.html    ← unstaged, but file still has changes
```

> Use this when: You committed too soon and want to re-organize what goes into the commit.

---

### `git reset --hard HEAD~1` — Undo commit AND discard all changes ⚠️

```bash
git reset --hard HEAD~1
```

**What changes:**
- `HEAD` moves back to `C2`
- `C3`'s changes are **permanently deleted** from files
- Working directory is reset to match `C2`

```bash
git status
# nothing to commit, working tree clean   ← completely clean

cat footer.html
# footer.html is back to how it was at C2 — the extra line is GONE
```

> ⚠️ **DANGER**: This is irreversible through normal Git commands. The changes are gone.

**From your session:**
```bash
git commit -am "added about us section"
# Oops! Committed by mistake

git reset --hard HEAD~1
# ← That entire commit is erased as if it never happened
```

---

### Visual Comparison

```
History:  C1 ──► C2 ──► C3 (HEAD)

git reset --soft HEAD~1:
  C1 ──► C2 (HEAD)     C3 changes → STAGED

git reset HEAD~1:
  C1 ──► C2 (HEAD)     C3 changes → UNSTAGED (modified files)

git reset --hard HEAD~1:
  C1 ──► C2 (HEAD)     C3 changes → DELETED (gone forever)
```

---

### `HEAD~N` Syntax

| Expression | Meaning |
|------------|---------|
| `HEAD` | Current commit |
| `HEAD~1` | One commit before current |
| `HEAD~2` | Two commits before current |
| `HEAD~3` | Three commits before |
| `4dd2a6d` | A specific commit by hash |

---

### Detached HEAD — Visiting Old Commits

From your session:
```bash
git checkout 4dd2a6d
```

**Output:**
```
Note: switching to '4dd2a6d'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, but any new commit you make can be lost unless
you create a branch.
```

**What this means:**
```
Before:  HEAD → main → C4
After:   HEAD → C2 (detached — not pointing to any branch!)
```

```bash
cat .git/HEAD
# 4dd2a6d3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9   ← raw hash, no branch name!
```

Go back to a branch:
```bash
git checkout main
cat .git/HEAD
# ref: refs/heads/main   ← attached again
```

---

> **Next Chapter →** [06_rebase.md](./06_rebase.md) — Rebasing for cleaner history
