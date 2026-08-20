# Day 1 — Git & GitHub Basics

## Why we use Git
- Tracks changes to code/files over time — full history, not just the latest version
- Lets you go back to any previous version if something breaks
- Enables collaboration — multiple people working on the same codebase without overwriting each other
- Local + remote backup (via GitHub) — your work isn't just sitting on one machine
- Industry standard — every DE/dev job expects Git literacy, no exceptions

## What is a Repo (Repository)
- A folder that Git is tracking — contains your project files + a hidden `.git/` folder
- `.git/` is where Git stores the entire history, branches, commits (the "database" of your project)
- Two types:
  - **Local repo** — lives on your machine
  - **Remote repo** — lives on GitHub (or similar), acts as the shared/backup copy
- A plain folder is NOT a repo until you run `git init`

## `git init`
- The first step — turns a plain folder into a Git repo
- Creates a hidden `.git/` folder → this is where all tracking/history lives
- Only local — does NOT connect to GitHub automatically (that's `git remote add origin <url>`, a separate step)
- Doesn't commit anything by itself — files exist in the folder, but nothing is tracked yet
- Run **once** per project

## Tracking states — how a file actually becomes "tracked"
Creating or editing a file does NOT mean Git is tracking it. Sequence:

**Untracked → Staged (`git add`) → Committed (`git commit`) → Tracked going forward**

- After `git init`, any new file you create shows as **untracked** — Git knows it exists, but records nothing about its content or changes
- Editing an untracked file (even multiple times) is invisible to Git — no history, no diff, nothing recoverable
- Once you `git add` a file, that exact content is frozen as a **staged snapshot** — it is NOT a live link to the file
  - If you edit the file again after `add` but don't `add` again, the staged snapshot still holds the OLD version
  - Committing at that point saves the old staged version, not your latest edit
  - You must re-run `git add` to update the staged snapshot before committing
- Only after `git commit` is a version permanently saved in history
- Once a file has been committed at least once, Git automatically detects future edits and shows them as "modified" — but brand new files always start as untracked until explicitly `add`ed

## `git status`
Shows the current state of working directory + staging area. Three buckets:
1. **Untracked files** — new, never staged/committed
2. **Changes not staged for commit** — tracked files edited since last `add`
3. **Changes to be committed** — staged (`add`ed), ready for next commit

A file can appear in bucket 2 and 3 at once if it was staged, then edited again afterward.

**Habit:** run `git status` before and after every `add`/`commit` — cheapest way to avoid committing the wrong thing.

## Staging shortcuts (for multiple changed files)
- `git add .` → stages everything (modified + new) from current folder downward
- `git add -A` → stages everything in the WHOLE repo (modified + new + deleted), regardless of which folder you're in
- `git add -u` → stages only already-tracked files (modified + deleted) — skips new/untracked files
- ⚠️ Caution: `add .` / `-A` will stage everything, including files you don't want in Git (`.env`, credentials, large CSVs) — set up `.gitignore` before using these freely

## `git add` → `git commit` → `git push` (the core daily chronology)
This is the loop you'll run constantly. Each step does something distinct — don't skip mentally to "push" as if it's one action.

**1. `git add <file>` (or `.` / `-A`)**
- Moves changes into the **staging area** — a frozen snapshot, not the file itself
- Local only, nothing saved to history yet
- Can be undone with `git restore --staged <file>` if you staged the wrong thing

**2. `git commit -m "message"`**
- Takes whatever is currently staged and saves it **permanently to local history**
- Each commit = a snapshot + message + timestamp + author, chained to the previous commit
- Still 100% local — nothing has left your machine yet
- Commit message discipline matters — this becomes part of your visible portfolio story on GitHub

**3. `git push`**
- Sends your local commits to the **remote repo** (GitHub)
- First time pushing a new local branch: `git push -u origin main` (`-u` sets upstream so future pushes can just be `git push`)
- After upstream is set, `git push` alone is enough
- If GitHub has commits you don't have locally (e.g., edited README on GitHub directly), push will be rejected — you'd need `git pull` first to sync

**Full everyday flow:**
```
git status              # see what changed
git add .               # stage it
git status               # confirm what's staged
git commit -m "message"  # save to local history
git push                 # send to GitHub
```

**Mental model:** `add` = pack the box → `commit` = seal and label the box (saved locally) → `push` = ship the box to GitHub (now backed up + visible remotely).

## Renamed / moved files
- If a file's path changes (e.g., folder restructured) but content is same, Git shows it as delete (old path) + untracked (new path)
- `git add -A` followed by `git status` will usually detect this correctly as a **rename**, not a real delete+create
- Always verify every "deleted" entry has a matching "untracked" counterpart before committing, to confirm nothing was actually lost

## LF vs CRLF warning
- **LF** (`\n`) — Linux/Mac line ending, Git's internal standard
- **CRLF** (`\r\n`) — Windows line ending
- Warning like `LF will be replaced by CRLF` = Git auto-converting endings per `core.autocrlf` setting — harmless, not an error
- On Windows, files are stored as LF in the repo but checked out as CRLF locally for compatibility

## `.gitignore`
- A plain text file listing patterns of files/folders Git should **never track**, even if you run `git add .`
- Lives at the repo root, named exactly `.gitignore`
- Each line = one pattern. Common syntax:
  - `.idea/` → ignore the whole folder
  - `*.log` → ignore all files with that extension, anywhere
  - `data/` → ignore a specific folder
  - `.env` → ignore a specific file
  - `!keep_this.csv` → exception, un-ignore a specific file inside an ignored pattern

**Why it matters for DE work specifically:**
- Credentials/keys (`.env`, `config.json` with secrets) — never belong in Git history, even private repos
- Large data files (`.csv`, `.parquet` datasets) — bloat repo size, GitHub isn't meant for data storage
- IDE/OS junk (`.idea/`, `.vscode/`, `.DS_Store`, `__pycache__/`) — machine-specific, not part of the actual project

**Important gotcha:** `.gitignore` only prevents tracking **new** files. If a file was already tracked (committed) before you added it to `.gitignore`, it will keep being tracked. You must explicitly untrack it first:
```
echo ".idea/" >> .gitignore
git rm -r --cached .idea
git add .gitignore
git commit -m "stop tracking IDE config files"
```
`git rm -r --cached` removes it from Git's tracking (and future commits) but leaves the actual file untouched on your disk.

**Starter `.gitignore` for a Python/DE repo:**
```
.idea/
.vscode/
__pycache__/
*.pyc
.env
*.csv
*.parquet
.DS_Store
```
(Adjust `*.csv` if you actually want small sample datasets tracked for a project — ignore raw/large data only.)

## `main` branch
- Created automatically at your **first commit** (not at `git init` — no branch exists until history exists)
- A branch = a movable pointer to a specific commit, nothing more
- `main` = default branch name (older repos used `master`) — represents the stable, always-working line of history
- Typical workflow: do risky/experimental work on a separate branch → merge back into `main` once it works → `main` stays stable throughout
- For solo learning repos, committing straight to `main` is fine for now
- For portfolio projects: feature branches + PRs merged into `main` is a strong signal to interviewers — shows real team workflow understanding

## What I'd forget in a week
1. `git init` = first step, creates `.git/`, does NOT connect to GitHub
2. Editing an untracked (or staged-but-not-re-added) file is invisible to Git until you `add` again
3. Staged snapshot is frozen at time of `add` — not a live link to the file
4. `git add -A` stages the whole repo including deletes; `git add .` only from current folder down
5. `add` = stage (local) → `commit` = save to local history → `push` = send to GitHub — three distinct steps, not one action
6. `.gitignore` only blocks NEW files from being tracked — already-tracked files need `git rm -r --cached` to actually stop tracking
7. `.idea/`, `.env`, large data files should always be in `.gitignore`, never tracked
8. CRLF/LF warning is harmless — just Git normalizing line endings on Windows
9. `main` branch is created at first commit, not at `git init`
10. Branches are just pointers — merging brings a branch's changes back into `main`

# JUST CHECKING
# Checking the issue from new branch