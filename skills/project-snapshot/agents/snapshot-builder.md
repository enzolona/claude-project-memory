# Snapshot Builder Agent

Specialized agent for one-shot generation of `snapshot.md`. Clean context. Synthesize the current project state by reading all available chronicles and vault indexes.

## Parameters received

`vault_path`, `repo_path`, `project_name`, `language`, `mode`, `date`

## Available tools

`Bash`, `Read`, `Write`

---

## Workflow

### 1 — Read existing indexes

```bash
cat "<vault_path>/Project/decisions-index.md" 2>/dev/null
cat "<vault_path>/Project/module-registry.md" 2>/dev/null
cat "<vault_path>/Project/dependency-graph.md" 2>/dev/null
cat "<vault_path>/Project/open-issues.md" 2>/dev/null
```

### 2 — Read chronicles in chronological order

```bash
ls "<vault_path>/Daily/"*.md 2>/dev/null | sort
```

For each chronicle, read only the sections relevant to the snapshot:

```bash
for f in $(ls "<vault_path>/Daily/"*.md | sort); do
  echo "=== $(basename $f) ==="
  awk '/^## (Summary|Decisions|Data \/ Storage|Code Changes)/{p=1} /^## [A-Z]/{if(!/^## (Summary|Decisions|Data \/ Storage|Code Changes)/)p=0} p{print}' "$f"
done
```

Give more weight to more recent chronicles — they are more representative of the current state. Older chronicles are useful for understanding the evolution and decisions still active.

### 3 — Analyze the repo structure

```bash
find "<repo_path>" -maxdepth 2 -type d \
  -not -path "*/.git/*" -not -path "*/.venv/*" \
  -not -path "*/node_modules/*" -not -path "*/__pycache__/*" | sort

# Dependency files
cat "<repo_path>/requirements.txt" 2>/dev/null || \
cat "<repo_path>/pyproject.toml" 2>/dev/null || \
cat "<repo_path>/package.json" 2>/dev/null || \
cat "<repo_path>/Cargo.toml" 2>/dev/null || \
cat "<repo_path>/go.mod" 2>/dev/null
```

### 4 — Write snapshot.md

Write `<vault_path>/Project/snapshot.md` with the structure below. Do not invent information not present in chronicles or the repo. If a section lacks sufficient data, write `<!-- TO COMPLETE -->` instead of guessing.

```markdown
# Project Snapshot — <project_name>

Current project state. Generated on <date> by reading <N> chronicles.
Updated automatically by daily-project-chronicle on structural changes.

---

## What the project does

*Generated from: <N> chronicles, latest [[<last_chronicle_date>-daily-project-chronicle]]*

<3–5 lines: project purpose, problem it solves, main output>

---

## Project structure

*Generated from: repo structure on <date>*

```
<folder structure — max 2 levels>
```

---

## Key modules

*Generated from: [[module-registry]] + chronicles*

| Module | Path | Role | Depends on |
|---|---|---|---|

---

## Active architectural decisions

*Generated from: [[decisions-index]]*

| Date | Decision | Impact |
|---|---|---|

---

## Input / Output

*Generated from: chronicles + repo structure*

**Input**: <format and path>
**Output**: <format and path>
**Artifacts**: <other files produced>

---

## Current status

*Generated on: <date>*

**Phase**: <active development | refactor | stabilization | production>
**Open issues**: <N from open-issues.md> — see [[open-issues]]
**Last significant change**: <date> — <description>

---

## Main external dependencies

*Generated from: repo dependency files*

| Dependency | Version | Use |
|---|---|---|
```

### Absolute rules

- The most recent date in the chronicles is the current state — not the oldest
- `DEFERRED` decisions in decisions-index do not go in active decisions
- `resolved` issues in open-issues do not go in current status
- Do not write generic prose or filler
- Use `Write` to create the file — this is a creation, not an update
