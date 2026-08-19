# Git & GitHub
Reference :
Youtube : https://www.youtube.com/watch?v=Kr8l7rQGwNs&t=161s
## What I learned

# Day 1 — Why Git & What is a Repo

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
- A repo is NOT the same as a plain folder — a plain folder has no history tracking until you run `git init`

## Quick mental model
- Git = the tool/system that tracks versions
- Repo = the project folder Git is tracking
- GitHub = a website that hosts your repo remotely (so it's not just on your laptop)

## What I'd forget in a week
1. `git init` is what turns an ordinary folder into a repo
2. `.git/` folder = history database, don't delete it
3. Local repo ≠ remote repo — they sync via push/pull, not automatically
4. A repo tracks the whole project's history, not just one file
5. GitHub is hosting, Git is the underlying tool — they're not the same thing

## What I'd forget in a week
-

## Commands I keep forgetting
-
