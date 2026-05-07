---
name: daily-project-chronicle
description: "Produce a dense end-of-day project chronicle for any software repository and saves it as an Obsidian daily note. Use when the user asks for a daily summary, end-of-day note, project chronicle, Obsidian daily note, code-change summary, test summary, or wants to log what happened today in the project. Trigger also on phrases like 'write today's chronicle', 'daily note', 'end of day summary', 'log today', 'what did we do today', 'write the note'."
---

# Daily Project Chronicle — Orchestrator

## Purpose

Generates a dense, accurate daily note for any git repository and saves it to the project's Obsidian vault. After writing, evaluates which downstream agents to activate based on today's changes.

---

## Step 1 — Read project config

```bash
cat "<vault_path>/project.config.yaml" 2>/dev/null
```

If `vault_path` is not known, ask the user. If the config exists, extract `project_name`, `repo_path`, `vault_path`, `language`. If missing or incomplete, go to Step 2.

---

## Step 2 — Collect parameters interactively (only if needed)

Ask the user once for any missing parameters.

| Parameter | Question | Default |
|---|---|---|
| `vault_path` | "Where is your Obsidian vault?" | required |
| `repo_path` | "Where is the project's git repository?" | required |
| `project_name` | "What is the project name?" | repo folder name |
| `language` | "Do you prefer notes in english or italiano?" | `english` |

After collecting, write or update `project.config.yaml`:

```yaml
project_name: <project_name>
repo_path: <repo_path>
vault_path: <vault_path>
language: <language>
```

Use `mkdir -p <vault_path>` if the vault does not exist yet.

---

## Step 3 — Extract additional parameters from the user's prompt

- **date**: note date. Default: today (`YYYY-MM-DD`). Use the user's date if specified (e.g. "yesterday").
- **focus_areas**: areas to focus on. Default: none.

---

## Step 4 — Collect structural diff

Before spawning the chronicler, collect the repo's structural diff. This will be used to decide which downstream agents to activate.

```bash
git -C <repo_path> diff --stat HEAD~1 HEAD 2>/dev/null || git -C <repo_path> diff --stat
git -C <repo_path> log --since="<date> 00:00" --until="<date> 23:59" \
  --name-status --no-merges --format="" 2>/dev/null
```

Save this output as `<structural_diff>` — a list of files with status A (added), M (modified), D (deleted), R (renamed).

---

## Step 5 — Spawn the Chronicler agent

Read `agents/chronicler.md` in the same directory as this skill, then spawn the sub-agent with:

```
project_name: <project_name>
vault_path: <vault_path>
repo_path: <repo_path>
language: <language>
date: <YYYY-MM-DD>
focus_areas: <focus_areas or "none">
```

**Note for dynamic session systems (e.g. Cowork):** if the repo path contains a session prefix that changes on every startup, resolve it first:

```bash
find /sessions/ -maxdepth 4 -name "<project_name>" -type d 2>/dev/null | head -1
```

---

## Step 6 — Evaluate downstream agents

When the chronicler completes, read the note just written and send Claude this evaluation prompt:

```
Given this daily note and structural diff, which agents should run?

STRUCTURAL DIFF:
<structural_diff>

DAILY NOTE (## Summary and ## Code Changes sections only):
<section_content>

Evaluate these agents:

1. decisions-agent: are there new decisions in the ## Decisions section?
2. module-registry-agent: are there new files/modules never seen before in ## Code Changes?
3. dependency-graph-agent: are there modified (M) or added (A) files in the diff?
4. snapshot-agent: are there STRUCTURAL changes? Triggers: added (A), renamed (R),
   or deleted (D) files; created/moved folders; output format changes;
   external dependencies added/removed.

Reply ONLY with this JSON, nothing else:
{
  "decisions": true/false,
  "module_registry": true/false,
  "dependency_graph": true/false,
  "snapshot": true/false,
  "reasons": {
    "decisions": "one line",
    "module_registry": "one line",
    "dependency_graph": "one line",
    "snapshot": "one line"
  }
}
```

---

## Step 7 — Run downstream agents

For each agent with value `true`, spawn it in sequence with:

```
vault_path: <vault_path>
repo_path: <repo_path>
project_name: <project_name>
language: <language>
date: <YYYY-MM-DD>
chronicle_path: <vault_path>/Daily/<date>-daily-project-chronicle.md
```

Agents and their files:

| Agent | File |
|---|---|
| dependency-graph-agent | `agents/dependency-graph-agent.md` |
| module-registry-agent | `agents/module-registry-agent.md` |
| decisions-agent | `agents/decisions-agent.md` |
| snapshot-agent | `agents/snapshot-agent.md` |

Execution order: `dependency-graph-agent` → `module-registry-agent` → `decisions-agent` → `snapshot-agent`. Snapshot always runs last because it can read the already-updated indexes.

---

## Step 8 — Report results

Tell the user:

1. Full path of the note written
2. Excerpt from `## Summary` (main bullets)
3. List of agents that ran with one-line reason for each
4. If `## Open Issues` has items in the note: "Found X new items — do you want to track them with manage-issues?"
