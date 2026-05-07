---
name: commit-semantic-extract
description: >
  Extracts semantic information from repository commit messages to answer
  questions about contributions to features, functional areas, or specific
  periods. On-demand skill: used only on explicit request, never automatically.
  Triggers: "what contributed to feature X", "tell me everything we did on this
  area", "extract semantics from commits", "commits related to", "what did we do
  on X in the last N days", "semantic commit extract", "history of feature X",
  "commit history for".
---

# Commit Semantic Extract — Orchestrator

## Purpose

Queries commit messages to answer semantic questions that chronicles don't cover directly. Does not write anything to the vault — responds in chat.

---

## Step 1 — Read config

```bash
cat "<vault_path>/project.config.yaml" 2>/dev/null
```

If missing, ask for `repo_path` and `vault_path`.

---

## Step 2 — Collect parameters from the query

From the user's message, extract:

- **query**: what they want to know (e.g. "authentication feature", "pipeline module")
- **date_range**: period of interest. Default: last 30 days.
- **output_format**: "commit list" or "narrative summary". Default: narrative summary.

---

## Step 3 — Run the extraction

Read `agents/semantic-extractor.md` in the same directory, then spawn with:

```
repo_path: <repo_path>
vault_path: <vault_path>
query: <query>
date_range: <date_range>
output_format: <output_format>
language: <language>
```

---

## Step 4 — Report results

Return the results in chat. Do not create vault files. At the end, if the results are relevant, suggest: "Do you want me to add this information to the snapshot as a historical note?"
