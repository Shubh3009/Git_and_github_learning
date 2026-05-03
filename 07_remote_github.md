# 📘 Chapter 7 — GitHub, Remote Repos, SSH & Push/Pull

> **Scenario**: Your website code is ready locally. Now you want to put it on GitHub  
> so the world can see it and so you can collaborate with others.

---

## What is a Remote?

A **remote** is a version of your repository hosted on a server (GitHub, GitLab, etc.).

```
YOUR MACHINE              GITHUB SERVER
┌────────────┐            ┌─────────────────────────────┐
│  local     │  push ──►  │ github.com/Shubh3009/repo   │
│  repo      │  ◄── pull  │                             │
└────────────┘            └─────────────────────────────┘
```

`origin` is just the **conventional nickname** for your primary remote.

---

## PART A — Connecting to GitHub

### Step 1: Create a new local repo

```bash
cd gitthree
git init
git status
```
```
On branch master
No commits yet
```

### Step 2: Create and commit files

```bash
# index.html already exists
git add .
git commit -m "add index.html"

git log --oneline
# a1b2c3d  add index.html
```

### Step 3: Add GitHub as the remote

```bash
git remote add origin https://github.com/Shubh3009/Rough_github.git
```

**What this does:**
- Creates an entry in `.git/config` under `[remote "origin"]`
- Does NOT push anything yet — just registers the address

```bash
git remote -v
```
**Output:**
```
origin  https://github.com/Shubh3009/Rough_github.git (fetch)
origin  https://github.com/Shubh3009/Rough_github.git (push)
```

Two lines — one for downloading (fetch), one for uploading (push). Usually the same URL.

### Step 4: Change Remote URL (if needed)

```bash
git remote set-url origin https://github.com/Shubh3009/Rough_github.git

# OR switch to SSH
git remote set-url origin git@github-personal:Shubh3009/Rough_github.git
```

### Remove a remote entirely

```bash
git remote remove origin
```

---

## PART B — SSH Authentication

### Why SSH instead of HTTPS?

| | HTTPS | SSH |
|--|-------|-----|
| Auth method | Username + password/token | Public/private key pair |
| Password needed? | Every push | Never (after setup) |
| Multiple accounts | Hard to manage | Easy with SSH config |

### How SSH Keys Work

```
YOUR MACHINE                    GITHUB
┌──────────────────┐            ┌──────────────────┐
│ Private Key      │            │ Public Key       │
│ ~/.ssh/id_ed25519│  ←match→  │ (added in GitHub │
│ (NEVER share!)   │            │  Settings)       │
└──────────────────┘            └──────────────────┘

When you push: GitHub challenges you → your private key responds → match → authenticated!
```

### View Your Public Key (to paste into GitHub)

```bash
cat ~/.ssh/id_ed25519_personal.pub
```
**Output:**
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB... shubham@macbook
```

Copy this entire output → paste into GitHub → Settings → SSH Keys → New SSH Key.

### Managing Multiple GitHub Accounts with SSH Config

```bash
nano ~/.ssh/config
```

Your `~/.ssh/config` file:
```
# Personal GitHub account
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal

# Work GitHub account
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
```

**How the alias works:**
- `github-personal` is your custom alias (can be anything)
- When Git sees `git@github-personal:...`, it replaces `github-personal` with `github.com` and uses the specified key

### Test your SSH connection

```bash
ssh -T git@github-personal
```
**Output:**
```
Hi Shubh3009! You've successfully authenticated, but GitHub does not provide shell access.
```

### Set Remote to Use SSH Alias

```bash
git remote set-url origin git@github-personal:Shubh3009/Rough_github.git
#                           ↑ your SSH config alias
#                                             ↑ username/repo on GitHub
git remote -v
# origin  git@github-personal:Shubh3009/Rough_github.git (fetch)
# origin  git@github-personal:Shubh3009/Rough_github.git (push)
```

---

## PART C — Push, Pull, Fetch, Clone

### `git push` — Send local commits to GitHub

**First push (sets upstream tracking):**
```bash
git push -u origin main
```

**Output:**
```
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 280 bytes | 280.00 KiB/s, done.
To github.com:Shubh3009/Rough_github.git
 * [new branch]      main -> main
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

**What `-u` does:**
- `-u` = `--set-upstream`
- Links your local `main` to `origin/main`
- After this, just `git push` works (no need to type `origin main` again)

**Subsequent pushes:**
```bash
git add .
git commit -m "add list"
git push           # ← no need to specify origin main anymore!
```

### Push a Feature Branch

```bash
git push -u origin navbar
```
This creates the `navbar` branch on GitHub too.

---

### `git clone` — Download a repo from GitHub

```bash
git clone https://github.com/hiteshchoudhary/golang.git
```

**What happens:**
1. Creates a folder `golang/` in your current directory
2. Downloads ALL files, ALL history, ALL branches
3. Automatically sets `origin` to point to the GitHub URL
4. Creates local `main` branch tracking `origin/main`

```bash
ls
# golang/     ← new folder created

cd golang
git remote -v
# origin  https://github.com/hiteshchoudhary/golang.git (fetch)
# origin  https://github.com/hiteshchoudhary/golang.git (push)

git log --oneline
# (shows entire commit history of that repo)
```

---

### `git fetch` — Download, but don't merge yet

```bash
git fetch
```

**What happens:**
- Downloads all new commits from `origin` into `origin/main` (tracking branch)
- Does NOT touch your working files
- Safe to run anytime — just "checks what's new"

```bash
# After fetch, see what's different
git diff main origin/main    # compare your local vs remote
git merge origin/main        # then manually merge when ready
```

---

### `git pull` — Fetch + Merge in one step

```bash
git pull
```

**Equivalent to:**
```bash
git fetch
git merge origin/main
```

**From your session:**
```bash
cd golang
git pull
# Already up to date.   ← nothing new on remote

git fetch
# (downloads any new commits silently)
```

### Fetch vs Pull

| | `git fetch` | `git pull` |
|--|------------|-----------|
| Downloads? | ✅ Yes | ✅ Yes |
| Updates working files? | ❌ No | ✅ Yes |
| Safe to run anytime? | ✅ Yes | ⚠️ May cause conflicts |
| Best for | Checking what's new | Updating your local copy |

---

## PART D — Git Config

### Why Config Matters

Every commit is signed with your name and email. GitHub uses the email to link commits to your account.

```bash
git config user.name     # see current name
git config user.email    # see current email
```

### Set Global Identity

```bash
git config --global user.name "Shubham Kumar"
git config --global user.email "shubham@example.com"
```

This writes to `~/.gitconfig`:
```
[user]
    name = Shubham Kumar
    email = shubham@example.com
```

### View All Config

```bash
git config --list
```
```
user.name=Shubham Kumar
user.email=shubham@example.com
core.repositoryformatversion=0
core.filemode=true
remote.origin.url=git@github-personal:Shubh3009/Rough_github.git
branch.main.remote=origin
branch.main.merge=refs/heads/main
```

### Show Config With Its Source File

```bash
git config --list --show-origin
```
```
file:/Library/Developer/.../gitconfig   core.autocrlf=input
file:/Users/shubhamkumar/.gitconfig     user.name=Shubham Kumar
file:/Users/shubhamkumar/.gitconfig     user.email=shubham@example.com
file:.git/config                        remote.origin.url=...
```

**Priority order (highest wins):**  
`local (.git/config)` > `global (~/.gitconfig)` > `system (/Library/.../gitconfig)`

---

> **Next Chapter →** [08_opensource.md](./08_opensource.md) — Open Source Contribution Workflow
