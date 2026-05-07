---
name: init-dependency-graph
description: >
  Scans the entire repository and builds the complete internal dependency map
  between files. One-shot skill to run once on existing projects. After
  initialization, the dependency graph is updated incrementally by
  daily-project-chronicle. Triggers: "initialize the dependency graph",
  "scan dependencies", "build dependency graph", "map project dependencies",
  "init dependency graph".
---

# Init Dependency Graph — Orchestrator

## Purpose

Builds `dependency-graph.md` by scanning the entire repo once. This is the full-scan version of the incremental update done by the chronicler.

---

## Step 1 — Read config

```bash
cat "<vault_path>/project.config.yaml" 2>/dev/null
```

If missing or incomplete, ask for `vault_path`, `repo_path`, `project_name`.

---

## Step 2 — Warn the user about timing

Before proceeding, say:
"I'll scan the entire repository to build the dependency map. On large repos (200+ files) this may take a few minutes."

---

## Step 3 — Spawn the Dependency Scanner agent

Read `agents/dependency-scanner.md` in the same directory, then spawn with:

```
vault_path: <vault_path>
repo_path: <repo_path>
project_name: <project_name>
date: <today YYYY-MM-DD>
mode: full-scan
```

---

## Step 4 — Report results

Tell the user:

1. Confirmation that `dependency-graph.md` was created
2. Number of files scanned and dependencies found
3. The 3–5 files with the most inbound dependencies (highest cascade impact)
4. "From now on the graph will be updated automatically by daily-project-chronicle"
