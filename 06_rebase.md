# 📘 Chapter 6 — Git Rebase

> **Scenario**: `bugfix` branch was created from an old commit on `main`.  
> Meanwhile `main` got new commits. You want `bugfix` to have those new commits too — without a messy merge commit.

---

## What is Rebase?

`git rebase` **replays your branch's commits on top of another branch**.  
It gives you a clean, linear history — as if you started your branch from the latest commit.

---

## Merge vs Rebase — The Core Difference

### Merge creates a "merge commit":
```
main:    A ──► B ──► C ──────────────► M (ugly merge commit)
                      \              /
bugfix:                D ──► E ──► F
```
History looks tangled.

### Rebase makes history linear:
```
main:    A ──► B ──► C ──► D' ──► E' ──► F'
                            ↑
                   (bugfix commits replayed here)
```
History looks like one clean line — as if you never branched.

> **D'**, **E'**, **F'** are NEW commit objects (different SHA hashes) with the same changes as D, E, F.

---

## Full Demo — Rebase `bugfix` onto `main`

### Setup: Create the diverged state

```bash
# On main — add pricing card
cat >> index.html << EOF
    pricing card added
EOF
git commit -am "pricing card added"

# Create bugfix from OLDER commit (before pricing card)
git checkout -b bugfix

# On bugfix — fix the about us section
cat >> index.html << EOF
    about us fixed
EOF
git commit -am "about us fixed"
```

**Current state:**
```
main:    C1 ──► C2 (navbar) ──► C3 (pricing card)
                  \
bugfix:            C4 (about us fixed)
```

```bash
# Check logs on each branch
git checkout main
git log --oneline
# C3  pricing card added
# C2  add navbar to code base
# C1  add index.html

git checkout bugfix
git log --oneline
# C4  about us fixed
# C2  add navbar to code base        ← bugfix doesn't know about C3!
# C1  add index.html
```

---

### Perform the Rebase

```bash
git checkout bugfix     # make sure you're on bugfix
git rebase main
```

**Git does this internally:**
1. Finds common ancestor of `bugfix` and `main` → that's `C2`
2. Saves `bugfix`'s commits after `C2` → saves `C4` (about us fixed)
3. Resets `bugfix` to match `main` tip → `C3` (pricing card)
4. Re-applies saved commits one by one → replays `C4` as `C4'`

**Output (clean rebase):**
```
Successfully rebased and updated refs/heads/bugfix.
```

**After rebase:**
```
main:    C1 ──► C2 ──► C3 (pricing card)
                              \
bugfix:                        C4' (about us fixed) ← replayed on top!
```

```bash
git log --oneline
# C4'  about us fixed
# C3   pricing card added   ← bugfix now includes main's work!
# C2   add navbar to code base
# C1   add index.html
```

---

## Rebase With Conflicts

Sometimes the replayed commits conflict with the base branch.  
From your session, this happened with `index.html`:

```bash
git rebase main
```

**Output:**
```
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
error: could not apply a1b2c3d... card added to index file
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
```

**`index.html` shows conflict markers:**
```html
<body>
    pricing card added
<<<<<<< HEAD
    images are being added to the project    ← from main
=======
    card added to index file                 ← from bugfix
>>>>>>> a1b2c3d (card added to index file)
</body>
```

### Resolve and continue:

```bash
# Step 1: Edit index.html manually — keep both lines
# <body>
#     pricing card added
#     images are being added to the project
#     card added to index file
# </body>

# Step 2: Stage the resolved file
git add index.html

# Step 3: Continue the rebase
git rebase --continue
```

**Output:**
```
[detached HEAD a2b3c4d] card added to index file
Successfully rebased and updated refs/heads/bugfix.
```

If you want to **abort the rebase entirely** and go back:
```bash
git rebase --abort
```

---

## Your Full Rebase Session Reconstructed

```bash
git checkout bugfix
git commit -am "card added to index file"     # commit on bugfix

git rebase main                               # conflict!
git add index.html                            # resolve conflict
git rebase --continue                         # finish rebase

git rebase main                               # run again to confirm clean
# Output: Current branch bugfix is up to date.
```

---

## Rebase vs Merge — When to Use Which?

| | Merge | Rebase |
|--|-------|--------|
| **History** | Non-linear (merge commits) | Linear (clean) |
| **Safety** | Safe on shared branches | ⚠️ Never rebase shared branches |
| **Use for** | Integrating finished features | Cleaning local feature branches |
| **Conflict** | Resolve once in merge commit | Resolve per commit replayed |

### The Golden Rule of Rebase

> ⚠️ **Never rebase a branch that others are working on!**
> 
> Rebase **rewrites commit history** (new SHA hashes). If your teammate based their work on the old hashes, it breaks their history.
>
> ✅ Safe: Rebase your **local feature branch** before opening a PR  
> ❌ Unsafe: Rebase `main` or any shared branch

---

> **Next Chapter →** [07_remote_github.md](./07_remote_github.md) — GitHub, SSH, Push & Pull
