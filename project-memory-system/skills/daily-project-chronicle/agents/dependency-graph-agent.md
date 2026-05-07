# Dependency Graph Agent

Specialized agent for incrementally updating `dependency-graph.md`. Clean context. Work only on files modified today — do not scan the entire repo.

## Parameters received

`vault_path`, `repo_path`, `project_name`, `language`, `date`, `chronicle_path`

## Available tools

`Bash`, `Read`, `Edit`, `Write`

---

## Import patterns by language

| Extension | Language | Pattern |
|---|---|---|
| `.py` | Python | `^import\s+([\w.]+)` and `^from\s+([\w.]+)\s+import` |
| `.js` `.jsx` `.ts` `.tsx` | JavaScript/TypeScript | `require\(['"]([^'"]+)['"]\)` and `from\s+['"]([^'"]+)['"]` |
| `.rs` | Rust | `^use\s+([\w:]+)` |
| `.java` | Java | `^import\s+([\w.]+)` |
| `.go` | Go | `"([^"]+)"` inside `import` block |
| `.rb` | Ruby | `require\s+['"]([^'"]+)['"]` |
| `.php` | PHP | `require\|include.*['"]([^'"]+)['"]` |
| `.cpp` `.h` | C/C++ | `#include\s+["<]([^">]+)[">]` |

---

## Workflow

### 1 — Identify files modified today

```bash
awk '/^## Code Changes/{p=1} /^## [A-Z]/{if(!/^## Code Changes/)p=0} p{print}' "<chronicle_path>" \
  | grep -oP '\[.*?\]\(\K[^)]+(?=\))'
```

### 2 — Read dependency-graph.md

```bash
cat "<vault_path>/Project/dependency-graph.md"
```

If it doesn't exist, create it:

```markdown
# Dependency Graph — <project_name>

Internal dependency map between project files.
Updated incrementally on every daily chronicle.
For the initial full scan, use the `init-dependency-graph` skill.

Format: `source_file --> dependency [type]`
Types: `imports` | `calls` | `extends` | `implements` | `uses`

## Dependencies

| File | Depends on | Type | Updated |
|---|---|---|---|
```

### 3 — Extract dependencies from each modified file

```bash
# Example for Python — adapt the pattern to the file's language
grep -nE "^import\s+|^from\s+" "<repo_path>/<file_path>"
```

For each import found:

1. Extract the referenced module/file name
2. Check if a file with that name exists in the repo:

```bash
find "<repo_path>" -name "<module_name>.*" \
  -not -path "*/.venv/*" \
  -not -path "*/__pycache__/*" \
  -not -path "*/node_modules/*" 2>/dev/null
```

3. Found → internal dependency → proceed
4. Not found → external library → skip

### 4 — Update the dependency registry

For each internal dependency found:

- If the row `<source_file> | <dependency_file>` **does not exist**: append it with today's date
- If the row **already exists**: update only the `Updated` column with today's date

Row format:

```
| <source_filename> | <dependency_filename> | imports | <date> |
```

### 5 — Add cascade impact to the chronicle

If dependencies were found, append this section to the chronicle under `## Code Changes` using `Edit`:

```markdown
### Cascade impact detected

| Modified file | Affects | Reason |
|---|---|---|
| <modified_file> | <dependent_file> | <dependent_file> imports <modified_file> |
```

Only add this section if relevant dependencies exist — do not add empty sections.

---

## Absolute rules

- Never analyze files outside the repo (`.venv/`, `node_modules/`, etc.)
- Never record external dependencies (libraries, frameworks)
- Use `Edit` to modify — do not overwrite files
- If no internal dependencies are found, stop without modifying anything
