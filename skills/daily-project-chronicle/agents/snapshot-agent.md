# Snapshot Agent

Specialized agent for updating `snapshot.md`. Clean context. Activated only when structural changes are detected. Reads the indexes already updated by the other agents before writing.

## Parameters received

`vault_path`, `repo_path`, `project_name`, `language`, `date`, `chronicle_path`

## Available tools

`Bash`, `Read`, `Edit`, `Write`

---

## Workflow

### 1 — Read the updated indexes

Read these files in sequence — they have already been updated by the upstream agents and represent the most current state:

```bash
cat "<vault_path>/Project/snapshot.md"
cat "<vault_path>/Project/module-registry.md"
cat "<vault_path>/Project/dependency-graph.md"
cat "<vault_path>/Project/decisions-index.md"
cat "<vault_path>/Project/open-issues.md"
```

### 2 — Read today's chronicle

```bash
cat "<chronicle_path>"
```

Focus on: `## Summary`, `## Code Changes`, `## Data / Storage`, `## Decisions`.

### 3 — Identify structural changes

Compare today's chronicle against the current snapshot. Identify:

- Files or folders added, renamed, or deleted
- Changes to output format or destination
- New external dependencies
- Changes to the overall architecture

### 4 — Update the snapshot surgically

Update **only the snapshot sections affected** by today's changes. Do not rewrite untouched sections.

Use `Edit` to modify specific sections — do not overwrite the entire file.

For each updated section, update the `Last updated` field at the top of the section with today's date and a link to the chronicle:

```markdown
*Last updated: [[<date>-daily-project-chronicle]]*
```

---

## snapshot.md format

If `snapshot.md` does not exist, create it with this full structure. If it exists, update only the relevant sections.

```markdown
# Project Snapshot — <project_name>

Current project state. Updated automatically when structural changes are detected.
This is not a history summary — it is a snapshot of the current state.

Generated: <date>
Last updated: [[<date>-daily-project-chronicle]]

---

## What the project does

*Last updated: [[<date>-daily-project-chronicle]]*

<3–5 line description of what the project does, its purpose, and the problem it solves>

---

## Project structure

*Last updated: [[<date>-daily-project-chronicle]]*

```
<main folder structure — max 2 levels deep>
```

---

## Key modules

*Last updated: [[<date>-daily-project-chronicle]]*

| Module | Path | Role | Depends on |
|---|---|---|---|
| <name> | <path> | <what it does in 1 line> | <main dependencies> |

---

## Active architectural decisions

*Last updated: [[<date>-daily-project-chronicle]]*

The most recent and relevant decisions that define the current architecture.
For the full list see [[decisions-index]].

| Date | Decision | Impact |
|---|---|---|
| <date> | <decision in 1 line> | <practical impact> |

---

## Input / Output

*Last updated: [[<date>-daily-project-chronicle]]*

**Input**: <format and path>
**Output**: <format and path>
**Artifacts**: <other files produced by the system>

---

## Current status

*Last updated: [[<date>-daily-project-chronicle]]*

**Phase**: <e.g. "active development", "refactor", "stabilization", "production">
**Open issues**: <N> — see [[open-issues]]
**Last significant change**: <date> — <1-line description>

---

## Main external dependencies

*Last updated: [[<date>-daily-project-chronicle]]*

| Dependency | Version | Use |
|---|---|---|
| <name> | <version> | <what it's used for> |
```

---

## Absolute rules

- Update only sections affected by today's changes
- Do not invent information not present in the chronicle or indexes
- Keep sections concise — the snapshot is a snapshot, not a log
- Every updated section must link to today's chronicle
- Use `Edit` — do not overwrite the entire file
