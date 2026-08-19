# DE-Prep

Single source of truth for my PL/SQL → Azure Data Engineering transition.
Code + notes live together, per topic. Commit small and often — this repo
doubles as my GitHub portfolio signal.

## Structure

- `01-git/` — Git & GitHub basics
- `02-python/` — Python core, OOP, DE-relevant modules (datetime, os, pathlib, json)
- `03-pandas/` — Pandas
- `04-sql/` — Advanced SQL
- `05-pyspark/` — PySpark basics
- `06-databricks-delta/` — Databricks Community Edition, Delta Lake
- `07-azure/` — ADLS Gen2, ADF, Azure orientation
- `08-ai-awareness/` — Lightweight notes on vector DBs, RAG, Azure OpenAI + data lakes
- `09-projects/` — The 3 portfolio projects (batch pipeline, incremental ADF pipeline, data quality pipeline)

Each topic folder has:
- `notes.md` — concepts, "what I'd forget in a week" bullets, gotchas
- `scripts/` — actual code I wrote while practicing

## Root files

- `weekly-review.md` — weekly review log
- `error-log.md` — running log of errors hit and how I fixed them
- `skill-checklist.md` — master checklist across the whole roadmap

## Daily habit

1. Write/practice code → save in the right `scripts/` folder
2. Add 3–5 bullets to that topic's `notes.md` answering "what would I forget in a week?"
3. `git add . && git commit -m "Day X: <topic>" && git push`
