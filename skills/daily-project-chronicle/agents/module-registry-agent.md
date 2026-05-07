# Module Registry Agent

Specialized agent for updating `module-registry.md`. Clean context. Work only on today's chronicle and the registry file.

## Parameters received

`vault_path`, `repo_path`, `project_name`, `language`, `date`, `chronicle_path`

## Available tools

`Bash`, `Read`, `Edit`, `Write`

---

## Workflow

### 1 — Read the Code Changes section from today's chronicle

```bash
awk '/^## Code Changes/{p=1} /^## [A-Z]/{if(!/^## Code Changes/)p=0} p{print}' "<chronicle_path>"
```

Extract all files mentioned in Markdown links: `[filename.ext](relative/path)`.

### 2 — Read module-registry.md

```bash
cat "<vault_path>/Project/module-registry.md"
```

If it doesn't exist, create it:

```markdown
# Module Registry — <project_name>

Registry of all key modules and files in the project.
Updated automatically when a new file appears in Code Changes.

| File | Path | Subsystem | Introduced | Description | Chronicle |
|---|---|---|---|---|---|
```

### 3 — Identify new files

Compare the files from today's chronicle against those already in the registry. A file is new if it does not appear in the `Path` column.

### 4 — Append new files

For each new file, append a row:

```
| <filename.ext> | <relative/path> | <subsystem> | <date> | <1-line description> | [[<date>-daily-project-chronicle]] |
```

To determine the subsystem, use the file path:
- `tests/`, `test_` prefix → `tests`
- `config/`, `*.yaml`, `*.toml`, `*.json` at root → `config`
- `scripts/` → `scripts`
- Otherwise: use the parent folder name as subsystem

For the description, briefly inspect the file:

```bash
head -20 "<repo_path>/<relative_path>"
```

Rules:
- Add only new files — do not modify existing rows.
- If all of today's files are already in the registry, stop without modifying anything.
- Use `Edit` to append — do not overwrite the file.
