---
name: manage-issues
description: >
  Manage project issues in the Obsidian vault: add new issues, update status,
  mark as resolved, list open or filtered issues. Use when the user says:
  "add issue", "track this problem", "mark as resolved", "list open issues",
  "what problems do we have open", "close the issue", "update the status of",
  "mark as in progress", "open issues", "track this bug".
---

# Manage Issues — Orchestrator

## Purpose

Manages the `open-issues.md` file in the project vault. Each issue has a unique ID, a status, a link to the origin chronicle, and (if resolved) a link to the resolution chronicle.

---

## Step 1 — Read config

```bash
cat "<vault_path>/project.config.yaml" 2>/dev/null
```

If `vault_path` is not known, ask the user. Extract `project_name` and `language`.

---

## Step 2 — Identify the requested operation

Classify the user's request into one of these types:

| Operation | Examples |
|---|---|
| **ADD** | "add issue", "track this problem", "I found a bug" |
| **UPDATE** | "update the status of", "mark as in progress" |
| **RESOLVE** | "mark as resolved", "close the issue", "fixed" |
| **LIST** | "list open issues", "what problems do we have", "show issues" |
| **DETAIL** | "tell me more about issue X", "details on ISSUE-007" |

---

## Step 3 — Spawn the Issue Manager agent

Read `agents/issue-manager.md` in the same directory as this skill, then spawn the sub-agent with:

```
operation: <ADD|UPDATE|RESOLVE|LIST|DETAIL>
vault_path: <vault_path>
project_name: <project_name>
language: <language>
user_input: <original user text with all details>
today: <YYYY-MM-DD>
```

---

## Step 4 — Report results

After the operation, tell the user:

- **ADD**: "Issue ISSUE-<id> added: <title>"
- **UPDATE**: "Issue ISSUE-<id> updated to status: <new_status>"
- **RESOLVE**: "Issue ISSUE-<id> marked as resolved on <date>"
- **LIST**: table of issues with status, title, and open date
- **DETAIL**: full detail of the requested issue
