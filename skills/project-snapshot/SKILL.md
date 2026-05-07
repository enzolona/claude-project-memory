---
name: project-snapshot
description: >
  Generates the current project state by synthesizing all available chronicles
  in the vault. One-shot skill to run once on existing projects, or to regenerate
  the snapshot if it is lost or corrupted. Triggers: "generate project snapshot",
  "create snapshot", "initialize snapshot", "build snapshot".
  Do NOT use for incremental updates — those are handled automatically by
  daily-project-chronicle.
---

# Project Snapshot — Orchestrator

## Purpose

Generates `snapshot.md` by reading all available chronicles and vault indexes. One-shot skill: produces a coherent snapshot of the current project state from accumulated history.

---

## Step 1 — Read config

```bash
cat "<vault_path>/project.config.yaml" 2>/dev/null
```

If missing or incomplete, ask interactively for `vault_path`, `repo_path`, `project_name`, `language`.

---

## Step 2 — Check prerequisites

```bash
ls "<vault_path>/Daily/"*.md 2>/dev/null | wc -l
ls "<vault_path>/Project/"*.md 2>/dev/null
```

If no chronicles exist, warn the user:
"No chronicles found in the vault. Do you want me to generate a minimal snapshot based only on the repo structure?"

If confirmed, proceed to Step 3 with `mode: repo-only`.

---

## Step 3 — Spawn the Snapshot Builder agent

Read `agents/snapshot-builder.md` in the same directory, then spawn the sub-agent with:

```
vault_path: <vault_path>
repo_path: <repo_path>
project_name: <project_name>
language: <language>
mode: full
date: <today YYYY-MM-DD>
```

---

## Step 4 — Report results

Tell the user:

1. Confirmation that `snapshot.md` was created with its full path
2. Excerpt from "What the project does" and "Current status" sections
3. Number of chronicles read to generate it
4. Any gaps detected: "I couldn't find information about X — consider adding it manually to the snapshot"
