# Lint Agent

Specialized agent for vault health checks. Clean context. Find problems, auto-fix mechanical ones, flag semantic ones.

## Parameters received

`vault_path`, `repo_path`, `project_name`, `language`, `date`

## Available tools

`Bash`, `Read`, `Edit`, `Write`

---

## Problem categories

| Category | Type | Action |
|---|---|---|
| Broken link to non-existent chronicle | mechanical | auto-fix |
| Broken link to non-existent issue | mechanical | auto-fix |
| Orphan page (file in registry never mentioned in any chronicle) | mechanical | auto-fix (add note) |
| Incorrect ISSUE-COUNT in open-issues.md | mechanical | auto-fix |
| Contradiction between snapshot and decisions-index | semantic | flag |
| Contradiction between two decisions in decisions-index | semantic | flag |
| Stale claim in snapshot (structure no longer exists in repo) | semantic | flag |
| Module in registry with a path that no longer exists in repo | semantic | flag |

---

## Workflow

### CHECK 1 — Broken links in chronicles

```bash
grep -roh '\[\[.*\]\]' "<vault_path>/Daily/" | sort -u
```

For each link of type `[[YYYY-MM-DD-daily-project-chronicle]]`:

```bash
ls "<vault_path>/Daily/<date>-daily-project-chronicle.md" 2>/dev/null
```

For each link of type `[[ISSUE-NNN]]`:

```bash
grep "ISSUE-<NNN>" "<vault_path>/Project/open-issues.md" 2>/dev/null
```

**Auto-fix**: if the file/issue doesn't exist, replace the link with plain text + note `*(broken link — file not found)*`.

---

### CHECK 2 — Orphan pages in module-registry

```bash
awk -F'|' 'NR>4 {print $2}' "<vault_path>/Project/module-registry.md" | tr -d ' '
```

For each file in the registry, verify it's mentioned in at least one chronicle:

```bash
grep -rl "<filename>" "<vault_path>/Daily/" 2>/dev/null | wc -l
```

**Auto-fix**: if a registry file appears in no chronicle, add a note in the Chronicle column: `*(no chronicle — added by init)*`.

---

### CHECK 3 — Correct ISSUE-COUNT

```bash
grep -c "^| ISSUE-" "<vault_path>/Project/open-issues.md" 2>/dev/null
grep "ISSUE-COUNT" "<vault_path>/Project/open-issues.md"
```

**Auto-fix**: if the counter doesn't match the actual row count, update it with `Edit`.

---

### CHECK 4 — Stale claims in snapshot vs real repo

Read the `## Project structure` section from the snapshot:

```bash
awk '/^## Project structure/{p=1} /^## [A-Z]/{if(!/^## Project structure/)p=0} p{print}' \
  "<vault_path>/Project/snapshot.md"
```

For each folder or file mentioned, verify it exists:

```bash
ls "<repo_path>/<mentioned_path>" 2>/dev/null
```

**Semantic flag** for each element not found:

```
PROBLEM: Snapshot mentions "<path>" but it no longer exists in the repo.
SNAPSHOT SAYS: <current text>
ACTUAL REPO: <real structure found with ls>
QUESTION: Do you want me to update the snapshot with the current structure?
```

---

### CHECK 5 — Contradictions between snapshot and decisions-index

Read the `## Active architectural decisions` section from the snapshot and compare it against the last 10 rows of decisions-index.

Look for contradictions like:
- Snapshot says "we use X" but a recent decision says "migrated to Y"
- Snapshot describes a structure that a decision says was abandoned

**Semantic flag** for each contradiction found:

```
PROBLEM: Potential contradiction between snapshot and decisions-index.
SNAPSHOT SAYS: "<text>"
DECISIONS-INDEX SAYS: "<text>" (date: <date>)
QUESTION: Which version is correct and current?
```

---

### CHECK 6 — Registry modules with non-existent paths

```bash
awk -F'|' 'NR>4 {print $3}' "<vault_path>/Project/module-registry.md" | tr -d ' '
```

For each path in the registry:

```bash
ls "<repo_path>/<path>" 2>/dev/null
```

**Semantic flag** for each path not found:

```
PROBLEM: Module registry contains "<file>" with path "<path>" but the file
does not exist in the repo.
POSSIBLE CAUSES: renamed, moved, deleted.
QUESTION: Was the file renamed/moved (tell me the new path) or deleted?
```

---

## Final output

Write a report to `<vault_path>/Meta/lint-<date>.md`:

```markdown
# Vault Lint Report — <date>

## Auto-fixes applied

- <fix description 1>
- <fix description 2>

## Flags for human review

### FLAG-001
**Type**: <problem type>
**Problem**: <description>
**Source A**: <text/path>
**Source B**: <text/path>
**Question**: <direct question>

### FLAG-002
...

## Stats

- Chronicles scanned: <N>
- Links checked: <N>
- Problems found: <N>
- Auto-fixes applied: <N>
- Flags for review: <N>
```

Save the report, then return the content to the orchestrator.
