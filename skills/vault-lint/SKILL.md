---
name: vault-lint
description: >
  Runs a health check on the project vault, finding contradictions, stale claims,
  broken links, and orphan pages. Hybrid approach: auto-fix for mechanical problems,
  flag for human review for semantic problems. Run periodically (monthly or on demand).
  Triggers: "lint the vault", "health check the vault", "check the vault",
  "vault lint", "check vault health", "find contradictions in the vault",
  "vault check", "clean up the vault".
---

# Vault Lint — Orchestrator

## Purpose

Keeps the vault healthy over time. Finds and fixes mechanical problems automatically. Flags semantic problems that require human judgment.

---

## Step 1 — Read config

```bash
cat "<vault_path>/project.config.yaml" 2>/dev/null
```

If missing, ask for `vault_path` and `project_name`.

---

## Step 2 — Spawn the Lint agent

Read `agents/lint-agent.md` in the same directory, then spawn with:

```
vault_path: <vault_path>
repo_path: <repo_path>
project_name: <project_name>
language: <language>
date: <today YYYY-MM-DD>
```

---

## Step 3 — Present results

When the lint agent completes, present to the user:

1. **Auto-fixes applied** — list of problems resolved automatically
2. **Flags for review** — list of problems requiring your decision, each with:
   - Problem description
   - The two conflicting versions
   - Direct question: "Which of these is correct?"
3. **Stats**: "Vault health: X problems found, Y auto-fixed, Z to review"

After presenting the flags, wait for the user's answers before applying semantic corrections.
