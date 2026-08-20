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

## `.gitkeep`
- Git does **not** track empty folders — only files. If a folder has zero files in it, Git ignores it entirely, even after `git add .`
- `.gitkeep` is a convention (not a real Git feature) to work around this: you place an empty file named `.gitkeep` inside the folder you want to preserve, just so the folder has *something* in it for Git to track
- Not a special filename to Git — you could name it anything (`.gitkeep` is just the community-agreed convention so people recognize its purpose)

**Example — keeping an empty `data/raw/` folder structure in your repo:**
```
mkdir -p data/raw
New-Item data/raw/.gitkeep      # PowerShell equivalent of `touch`
git add data/raw/.gitkeep
git commit -m "add folder structure for raw data"
```
Now `data/raw/` will exist for anyone who clones your repo, even though it has no real data in it yet (real data likely stays gitignored anyway).

**When you'd actually use this:** scaffolding a project's folder structure (e.g., `data/raw/`, `data/processed/`, `logs/`) before any real files exist in those folders, so the structure itself is visible on GitHub as part of your portfolio project layout.

## Git branches
A branch is a **movable pointer to a commit** — lets you work on something without touching `main`.

**Create a branch (without switching to it):**
```
git branch feature-pyspark-notes
```

**Create AND switch to it in one step (most common):**
```
git checkout -b feature-pyspark-notes
```
or the newer syntax:
```
git switch -c feature-pyspark-notes
```

**List all branches (current one marked with `*`):**
```
git branch
```
Example output:
```
* main
  feature-pyspark-notes
  feature_chalke_1
```

**Switch between existing branches:**
```
git checkout feature-pyspark-notes
```
or
```
git switch feature-pyspark-notes
```

**Delete a branch once it's merged and no longer needed:**
```
git branch -d feature-pyspark-notes
```
(`-D` instead of `-d` force-deletes even if it's not merged — use carefully)

**Real example from practice:** created `feature_chalke_1`, made changes there, later needed to switch back to `main` — hit a block because `.idea/` files were untracked on `main` but tracked on `feature_chalke_1`, so Git refused the switch (see Merge Conflict section below for the fix).

## Git merge
Merging takes the changes from one branch and brings them into another — typically feature branch → `main`.

**Basic flow:**
```
git checkout main                        # go to the branch you want to merge INTO
git merge feature-pyspark-notes          # bring in changes FROM this branch
```

**Two possible outcomes:**
1. **Fast-forward merge** — if `main` hasn't changed since you branched off, Git just moves `main`'s pointer forward to match your branch. No new merge commit, clean and simple.
2. **Merge commit** — if both branches have diverged (both have new commits), Git creates a new commit that combines both histories:
```
git merge feature_chalke_1 -m "Checking my 2nd Merge"
```
This `-m` message describes the merge itself, separate from the individual commits being merged.

**After merging, the feature branch's commits become part of `main`'s history** — you can safely delete the feature branch at this point if you're done with it.

## Git merge conflict
A conflict happens when Git **cannot automatically decide** how to combine changes — usually because the same file was changed differently on both branches, or one branch deleted a file the other modified.

**Example 1 — content conflict (same line changed differently on both branches):**
```
Auto-merging .idea/workspace.xml
CONFLICT (content): Merge conflict in .idea/workspace.xml
Automatic merge failed; fix conflicts and then commit the result.
```
Git couldn't pick between the two versions automatically. Fix: manually resolve, then stage + commit.
```
git checkout --ours .idea/workspace.xml   # or --theirs, depending on which version you want
git add .idea/workspace.xml
git commit
```

**Example 2 — modify/delete conflict (deleted on one branch, changed on the other):**
```
CONFLICT (modify/delete): .idea/workspace.xml deleted in HEAD and modified in feature_chalke_1.
Version feature_chalke_1 of .idea/workspace.xml left in tree.
```
This happened because the file was removed from tracking on `main`, but still tracked+modified on `feature_chalke_1`. Fix (decide to keep the deletion):
```
git rm .idea/workspace.xml
git commit
```

**Example 3 — blocked before merge even starts (untracked file conflict):**
```
error: The following untracked working tree files would be overwritten by checkout:
        .idea/workspace.xml
Please move or remove them before you switch branches.
```
Not technically a merge conflict, but the same family of issue — file state differs between branches enough that Git refuses to proceed automatically. Fix: remove/move the untracked file, or force the switch:
```
Remove-Item -Recurse -Force .idea
git switch feature_chalke_1
```

**How to actually resolve a real (non-`.idea`) content conflict:**
When Git can't auto-merge a real code/text file, it inserts conflict markers directly into the file:
```
<<<<<<< HEAD
value_from_main
=======
value_from_feature_branch
>>>>>>> feature-pyspark-notes
```
You manually edit the file to keep the correct version (delete the markers and the version you don't want), then:
```
git add <file>
git commit
```

**Root-cause pattern to remember:** most of the conflicts you'll hit early on (like the `.idea/` saga) aren't real code conflicts — they're caused by tracking files that should have been gitignored from the start. Real conflicts (in actual project code) are resolved the same way: pick/merge the correct content manually, `add`, `commit`.

## Fast-forward merge (main) — deeper look
- Happens when the branch you're merging INTO (`main`) has **no new commits** since you branched off — `main`'s history is a straight line, unchanged
- Git doesn't need to combine anything — it just moves `main`'s pointer forward to match your feature branch's latest commit
- No new merge commit is created — history stays linear, as if you'd committed directly to `main`

**Example:**
```
git checkout -b feature-x        # branch off main
# ...make 2 commits on feature-x...
git checkout main                # main hasn't moved at all
git merge feature-x
```
Output:
```
Updating a1b2c3d..e4f5g6h
Fast-forward
 file.txt | 2 ++
 1 file changed, 2 insertions(+)
```

**When it does NOT fast-forward:** if `main` also got new commits (e.g., someone else pushed, or you committed on `main` too) while you were working on `feature-x`, Git can't just move the pointer — it must create a real merge commit combining both histories (see "Git merge conflict" section above for what happens if those changes overlap).

**Force a real merge commit even when fast-forward is possible** (some teams prefer this for clearer history):
```
git merge feature-x --no-ff
```

## `git rebase`
Rebase takes your branch's commits and **replays them on top of another branch's latest commit** — rewrites history to look linear, instead of creating a merge commit.

**Basic example:**
```
git checkout feature-x
git rebase main
```
This takes all commits unique to `feature-x`, and replays them one by one on top of `main`'s current tip — as if you'd branched off from the latest `main`, not the old one.

**Merge vs rebase — the key difference:**
- `merge` → preserves exact history, creates a merge commit joining both branches (non-linear, shows the real story)
- `rebase` → rewrites your branch's commits to sit on top of the other branch (linear, cleaner-looking history, but changes commit hashes)

**⚠️ Golden rule:** never rebase commits that have already been pushed and shared with others (or across branches others use) — rebase rewrites commit history/hashes, which breaks things for anyone who already has the old commits. Safe on local/solo branches only.

**If a rebase hits conflicts** (same idea as merge conflicts):
```
git rebase main
# CONFLICT ...
# fix the file manually
git add <file>
git rebase --continue
```
Abort entirely if it gets messy:
```
git rebase --abort
```

## `git reflog`
A safety net — logs **every** movement of `HEAD` (checkouts, commits, resets, rebases, merges), even ones that aren't visible in `git log`. This is how you recover "lost" commits.

**Example:**
```
git reflog
```
Output:
```
e4f5g6h HEAD@{0}: commit: fixed pipeline script
a1b2c3d HEAD@{1}: reset: moving to HEAD~1
9f8e7d6 HEAD@{2}: commit: added notes.md
```

**Real use case — you `reset --hard` and regret it:**
```
git reflog                    # find the commit hash before your reset
git reset --hard <that-hash>  # restore back to it
```
`reflog` entries expire after ~90 days by default, but for anything recent, this is your undo-button-of-last-resort when a commit seems to have "disappeared."

## `git reset` — using HEAD and using commit ID
Reset moves your branch pointer (and optionally your staged/working files) to a different commit.

**Three modes (from least to most destructive):**
```
git reset --soft <target>     # move pointer only, keep changes staged
git reset --mixed <target>    # move pointer, unstage changes (default if you omit the flag)
git reset --hard <target>     # move pointer, discard changes completely
```

**Using HEAD-relative references:**
```
git reset --hard HEAD~1       # go back 1 commit from current
git reset --hard HEAD~3       # go back 3 commits
```

**Using a specific commit ID (from `git log` or `git reflog`):**
```
git log --oneline             # find the hash you want, e.g. 9f8e7d6
git reset --hard 9f8e7d6      # jump directly to that exact commit
```

**Practical example (matches what you did earlier):** removing your last mistaken commit that was never pushed:
```
git log --oneline -5          # confirm the mistaken commit is on top
git reset --hard HEAD~1       # gone completely
```

## `git diff`
Shows exact line-by-line changes — what's different between two states (working directory, staged area, commits, branches).

**Most common uses:**
```
git diff                      # working directory vs staged area (unstaged changes)
git diff --staged             # staged area vs last commit (what will be committed)
git diff HEAD                 # working directory vs last commit (everything changed, staged or not)
git diff main feature-x       # compare two branches
git diff <commit1> <commit2>  # compare two specific commits
```

**Example output:**
```
git diff
diff --git a/notes.md b/notes.md
--- a/notes.md
+++ b/notes.md
@@ -1,3 +1,4 @@
 # Day 1 notes
-old line
+new line
+another new line
```
`-` = removed, `+` = added. This is your go-to before `git add` to sanity-check exactly what you're about to stage.

## `git cherry-pick`
Takes a **specific single commit** from one branch and applies it onto your current branch — without merging the whole branch.

**Example:**
```
git checkout main
git log feature-x --oneline        # find the commit hash you want, e.g. b2c3d4e
git cherry-pick b2c3d4e
```
This applies just that one commit's changes onto `main`, as a new commit — the rest of `feature-x`'s commits are left out.

**When you'd use this:** you made 5 commits on a feature branch, but only 1 of them (a bugfix) is ready for `main` right now — the other 4 are still experimental. Cherry-pick just the bugfix commit instead of merging everything.

**If it conflicts** (same resolution pattern as merge/rebase):
```
git add <file>
git cherry-pick --continue
```

## Stashing (`git stash`)
Temporarily shelves uncommitted changes (staged or unstaged) so you can switch branches or pull cleanly, without committing half-finished work.

**Basic flow:**
```
git stash                     # shelve current changes, working directory becomes clean
# ...switch branches, pull, whatever you needed a clean state for...
git stash pop                 # bring the shelved changes back
```

**Useful variants:**
```
git stash list                # see all stashed sets (you can stash multiple times)
git stash apply                # reapply without removing it from the stash list (pop removes it, apply keeps it)
git stash drop                 # delete a stash you no longer need
git stash push -m "wip: adf notes"   # stash with a custom label, easier to identify later
```

**Real scenario:** you're mid-edit on `notes.md`, need to quickly switch to `main` to check something, but aren't ready to commit yet:
```
git stash
git checkout main
# ...do the thing...
git checkout feature-pyspark-notes
git stash pop
```

## `git push` and `git clone`

**`git push` — sending local commits to GitHub:**
```
git push                          # after upstream is already set
git push -u origin main           # first push of a new branch, sets upstream (-u) so future pushes are just `git push`
git push origin feature-x         # explicitly push a specific branch
git push --force                  # overwrite remote history (careful — only on solo/local branches, see reset section)
```

**`git clone` — copying a remote repo to your machine (the opposite direction of push):**
```
git clone https://github.com/<username>/<repo-name>.git
```
- Downloads the full repo (all history, all branches) into a new local folder named after the repo
- Automatically sets up the remote connection (`origin`) — no need to run `git remote add` separately
- Use this instead of `git init` when starting from an existing GitHub repo (e.g., cloning your own repo onto a new laptop, or a job's existing codebase)

**Clone into a specific folder name:**
```
git clone https://github.com/<username>/<repo-name>.git my-folder-name
```

**Difference to keep straight:**
- `git init` → start a brand-new repo from scratch, locally
- `git clone` → copy an already-existing remote repo to your machine

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
11. Git doesn't track empty folders — use `.gitkeep` to preserve folder structure
12. `git checkout -b <name>` = create + switch in one step; `git branch <name>` alone does NOT switch
13. Fast-forward merge = no diverged history, no merge commit; diverged branches = real merge commit needed
14. Merge conflict markers `<<<<<<<` `=======` `>>>>>>>` — manually edit, remove markers, then `add` + `commit`
15. Most early conflicts trace back to tracking files that should've been gitignored (like `.idea/`) — not real code conflicts
16. Fast-forward merge = no merge commit, just moves the pointer; happens only if `main` hasn't diverged
17. Rebase rewrites history (linear, clean) — never rebase commits already pushed/shared
18. `git reflog` shows every HEAD movement, even after `reset --hard` — your recovery tool when something seems "lost"
19. Reset modes: `--soft` (keep staged) → `--mixed` (unstage) → `--hard` (discard completely) — increasing destructiveness
20. `git diff` (unstaged) vs `git diff --staged` (staged) — check before every `add`
21. `cherry-pick` = grab one specific commit from another branch, not the whole branch
22. `git stash` = shelve uncommitted work temporarily, `stash pop` brings it back — use before switching branches mid-edit
23. `git clone` = copy an existing remote repo locally (sets up `origin` automatically); `git init` = start brand new