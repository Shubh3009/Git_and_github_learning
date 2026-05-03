# 🧑‍💻 Git & GitHub — Interactive Documentation
> By Shubham Kumar | Based on hands-on terminal session

All examples use real demo files: `index.html`, `nav-bar.html`, `footer.html`, `hero-section.html`

---

## 📚 Chapters

| # | Chapter | Key Topics |
|---|---------|-----------|
| 01 | [Git Basics](./docs/01_git_basics.md) | `git init`, `git status`, `git add`, `git commit` |
| 02 | [Branches](./docs/02_branches.md) | `git branch`, `git checkout`, `git checkout -b`, `-d` |
| 03 | [Log & Diff](./docs/03_log_and_diff.md) | `git log --oneline`, `git diff`, comparing commits |
| 04 | [Merge & Conflicts](./docs/04_merge.md) | Fast-forward, 3-way merge, conflict resolution |
| 05 | [Stash & Reset](./docs/05_stash_reset.md) | `git stash`, `git reset --hard`, detached HEAD |
| 06 | [Rebase](./docs/06_rebase.md) | `git rebase`, conflict during rebase, `--continue` |
| 07 | [Remote & GitHub](./docs/07_remote_github.md) | `git remote`, SSH config, `push`, `pull`, `clone` |
| 08 | [Open Source](./docs/08_opensource.md) | Fork → Clone → Branch → PR workflow |

---

## 🗂️ Demo Files Used

```
gittwo/
├── index.html         ← main project file, edited across all branches
├── nav-bar.html       ← built on nav-bar branch, merged into main
├── hero-section.html  ← built on main branch
└── footer.html        ← built on footer branch, caused merge conflict
```

---

## ⚡ Quick Command Reference

```bash
# BASICS
git init                     # start tracking a folder
git status                   # see what changed
git add <file>               # stage a file
git add .                    # stage everything
git commit -m "message"      # save snapshot
git commit -am "message"     # add tracked files + commit

# BRANCHES
git branch                   # list branches
git branch <name>            # create branch
git checkout <name>          # switch branch
git checkout -b <name>       # create + switch
git branch -d <name>         # delete (safe)

# HISTORY
git log --oneline            # compact history
git diff                     # unstaged changes
git diff --staged            # staged changes
git diff <hash1> <hash2>     # between commits

# MERGE & REBASE
git merge <branch>           # merge into current
git rebase <branch>          # rebase onto branch
git rebase --continue        # after conflict fix

# STASH
git stash                    # save WIP
git stash pop                # restore WIP

# UNDO
git reset --hard HEAD~1      # undo last commit (DANGER)
git reset HEAD~1             # unstage last commit

# REMOTE
git remote add origin <url>  # connect to GitHub
git remote -v                # list remotes
git remote set-url origin <url>
git push -u origin main      # first push
git push                     # subsequent pushes
git pull                     # fetch + merge
git clone <url>              # download repo
```
