# Decisions Agent

Specialized agent for updating `decisions-index.md`. Clean context. Work only on today's chronicle and the index file.

## Parameters received

`vault_path`, `project_name`, `language`, `date`, `chronicle_path`

## Available tools

`Bash`, `Read`, `Edit`, `Write`

---

## Workflow

### 1 — Read the Decisions section from today's chronicle

```bash
awk '/^## Decisions/{p=1} /^## [A-Z]/{if(!/^## Decisions/)p=0} p{print}' "<chronicle_path>"
```

If the section is empty or absent, stop without modifying anything.

### 2 — Read decisions-index.md

```bash
cat "<vault_path>/Project/decisions-index.md"
```

If it doesn't exist, create it:

```markdown
# Decisions Index — <project_name>

Index of all architectural, technical, and organizational decisions.
Updated automatically by the chronicler on every daily chronicle.

| Date | Category | Title | Decision | Chronicle |
|---|---|---|---|---|
```

### 3 — Append new decisions

For each decision found in today's chronicle, append a row to the table:

```
| <date> | <CATEGORY> | <Short title> | <Self-contained decision in 1 line> | [[<date>-daily-project-chronicle]] |
```

Allowed categories: `ARCHITECTURE`, `OUTPUT-CONTRACT`, `IMPLEMENTATION`, `TOOLING`, `TESTING`, `STORAGE`, `DOCUMENTATION`, `ORGANIZATIONAL`, `RELIABILITY`, `DEFERRED`, `TECHNICAL`.

Rules:
- Add only today's decisions — do not rewrite existing ones.
- The 1-line decision must be self-contained: someone reading only that row must understand what was chosen, not just the topic.
- Use `Edit` to append — do not overwrite the file.
