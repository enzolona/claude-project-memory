---
name: read-chronicles
description: >
  Read and query daily project chronicles saved in an Obsidian vault.
  Use when the user asks about project history, architectural decisions,
  evolution of a module or file, test errors, applied fixes, open issues,
  or what happened in a specific period. Trigger on phrases like:
  "what did we do", "when was X introduced", "when did it break",
  "find this error in the tests", "when was this fixed", "why did we choose X",
  "show me the evolution of", "what was blocked", "what happened last week",
  "summarize the project", "history of", "when did it change",
  "search the chronicles", "what contributed to".
---

# Read Chronicles

Skill for querying daily project chronicles from an Obsidian vault efficiently.
Read only the sections needed to answer the query.

---

## Step 1 — Read project config

```bash
cat "<vault_path>/project.config.yaml" 2>/dev/null
```

If `vault_path` is not known, ask the user. If the config exists, extract `vault_path`, `project_name`, `language`. If it doesn't exist, ask only for `vault_path`.

---

## Step 2 — Collect parameters from the query

From the user's message, extract:

- **query**: what they want to know (free text)
- **date_range**: period of interest
  - "recently", "lately", "this week" → last 7 days
  - specific date → that date only
  - event without date → search entire archive
  - no time reference → default last 14 days

---

## Step 3 — Identify query type

Classify the query into one of these types. The type determines which sections to read.

| Type | Examples | Primary sources |
|---|---|---|
| **STATUS** | "what did we do", "summarize", "last week" | `## Summary` + `## Next Steps` |
| **DECISION** | "why did we choose X", "when did we decide Y" | `decisions-index.md` → chronicle for detail |
| **MODULE_INTRO** | "when was X introduced", "when did it appear" | `module-registry.md` → `## Code Changes` |
| **MODULE_BROKEN** | "when did it stop working", "when did it break" | `## Summary` + `## Tests` + `## Reliability` |
| **TEST_ERROR** | "find this error in the tests", "when did this traceback appear" | `## Tests` (grep keyword) |
| **FIX** | "when was this fixed", "when were these fixes applied" | `## Code Changes` + `## Reliability` |
| **FILE_HISTORY** | "show me the evolution of file.py", "history of this file" | `module-registry.md` + grep chronicles + `git log` |
| **ISSUE** | "open issues", "known problems", "when was X resolved" | `open-issues.md` → chronicle for detail |
| **PERIOD** | "what happened between April 22 and 27" | all sections for that date range |

If the query spans multiple types, combine the corresponding patterns.

---

## Step 4 — Retrieve available files

```bash
ls "<vault_path>/Daily/"*.md 2>/dev/null | sort
```

Filter by `date_range`. For queries without a date, use all available files.

Index files (check before searching chronicles):

- `<vault_path>/Project/decisions-index.md`
- `<vault_path>/Project/module-registry.md`
- `<vault_path>/Project/open-issues.md`
- `<vault_path>/Project/snapshot.md`

---

## Step 5 — Efficient extraction by query type

Never read the full content of every chronicle. Use the following commands.

### STATUS — Summary + Next Steps

```bash
for f in <file_list>; do
  echo "=== $(basename $f) ==="
  awk '/^## Summary/{p=1} /^## [A-Z]/{if(!/^## Summary/)p=0} p{print}' "$f"
  awk '/^## Next Steps/{p=1} /^## [A-Z]/{if(NR>1 && !/^## Next Steps/)p=0} p{print}' "$f"
done
```

### DECISION — index first

```bash
cat "<vault_path>/Project/decisions-index.md"
```

If the table row is not sufficient, read the linked chronicle:

```bash
awk '/^## Decisions/{p=1} /^## [A-Z]/{if(!/^## Decisions/)p=0} p{print}' "<chronicle>"
```

### MODULE_INTRO — registry first

```bash
cat "<vault_path>/Project/module-registry.md"
grep -l "<module_or_file>" "<vault_path>/Daily/"*.md | sort | head -1
awk '/^## Code Changes/{p=1} /^## [A-Z]/{if(!/^## Code Changes/)p=0} p{print}' "<first_chronicle>"
```

### MODULE_BROKEN — search failures in Tests

```bash
grep -l "<module_or_keyword>" "<vault_path>/Daily/"*.md | sort
awk '/^## (Summary|Tests|Reliability)/{p=1} /^## [A-Z]/{if(!/^## (Summary|Tests|Reliability)/)p=0} p{print}' "<file>"
```

### TEST_ERROR — direct grep on keyword

```bash
grep -r -l "<error_keyword>" "<vault_path>/Daily/"
awk '/^## Tests/{p=1} /^## [A-Z]/{if(!/^## Tests/)p=0} p{print}' "<file>"
```

### FIX — Code Changes + Reliability

```bash
grep -l "<module_or_fix>" "<vault_path>/Daily/"*.md | sort
awk '/^## (Code Changes|Reliability)/{p=1} /^## [A-Z]/{if(!/^## (Code Changes|Reliability)/)p=0} p{print}' "<file>"
```

### FILE_HISTORY — combine registry, chronicles, and git

```bash
# 1. Check module-registry for introduction date
cat "<vault_path>/Project/module-registry.md" | grep "<filename>"

# 2. Search chronicles (narrative)
grep -rl "<filename>" "<vault_path>/Daily/" | sort

# 3. Git history (precise, with hashes)
git -C <repo_path> log --oneline -- <path/to/file>

# 4. For each matching chronicle, read Code Changes
awk '/^## Code Changes/{p=1} /^## [A-Z]/{if(!/^## Code Changes/)p=0} p{print}' "<chronicle>"
```

### ISSUE — open-issues first

```bash
cat "<vault_path>/Project/open-issues.md"
```

For detail on a specific issue, follow the link to the origin chronicle.

### PERIOD — all sections in range

```bash
for f in <file_list_in_range>; do
  echo "=== $(basename $f) ==="
  cat "$f"
done
```

---

## Step 6 — Response in chat

Reply directly in conversation. Do not create files. Match the response language to the `language` in config (default: English).

### FORMAT — STATUS

```
**[YYYY-MM-DD]** — <main fact of the day in 1 line>
**[YYYY-MM-DD]** — <main fact of the day in 1 line>
...

**Open items as of <most recent date>:**
- <item 1 from Next Steps>
- <item 2 from Next Steps>
```

### FORMAT — DECISION

```
**Decision:** <title>
**Date:** YYYY-MM-DD
**Choice:** <what was decided in 1–2 sentences>
**Why:** <main rationale>
**Discarded alternative:** <if present>
**Chronicle:** <link to file>
```

### FORMAT — MODULE_INTRO / FIX / MODULE_BROKEN

```
**First occurrence:** YYYY-MM-DD — <context>
**Commit:** <hash if available>
**Why:** <motivation from chronicle>
```

### FORMAT — TEST_ERROR

```
**Found on:** YYYY-MM-DD
**Context:** <what was being tested>
**Error:** <relevant excerpt>
**Resolved:** yes/no — <date and method if yes>
```

### FORMAT — FILE_HISTORY

```
**[YYYY-MM-DD]** `<hash>` — <change description>
**[YYYY-MM-DD]** `<hash>` — <change description>
...
```

Chronological order, oldest to newest. Use chronicles for the "why" and git for the "exactly when".

### FORMAT — ISSUE

```
**[ISSUE-<id>]** <title> — Status: <open|in_progress|resolved>
Opened: YYYY-MM-DD | Chronicle: <link>
Description: <text>
Resolution: <if resolved>
```

---

## General rules

- Never read a section that is not needed for the query.
- Indexes (`decisions-index.md`, `module-registry.md`, `open-issues.md`, `snapshot.md`) are always the first source — go to chronicles only if more detail is needed.
- Chronicles are narrative (the "why"); git is the source for "exactly when" and "what was in the code".
- If nothing is found in the chronicles, say so explicitly and suggest `git log` as an alternative.
- If the query is ambiguous across multiple types, choose the most likely type and state the assumption made.
