# Dependency Scanner Agent

Specialized agent for full-repo dependency scanning. Clean context. Scans all source files and builds `dependency-graph.md` from scratch.

## Parameters received

`vault_path`, `repo_path`, `project_name`, `date`, `mode`

## Available tools

`Bash`, `Read`, `Write`

---

## Import patterns by language

| Extension | Language | grep command |
|---|---|---|
| `.py` | Python | `grep -nE "^import\|^from" <file>` |
| `.js` `.jsx` | JavaScript | `grep -nE "require\(|^import" <file>` |
| `.ts` `.tsx` | TypeScript | `grep -nE "require\(|^import" <file>` |
| `.rs` | Rust | `grep -nE "^use " <file>` |
| `.java` | Java | `grep -nE "^import " <file>` |
| `.go` | Go | `grep -nE "\"" <file>` (inside import block) |
| `.rb` | Ruby | `grep -nE "^require" <file>` |
| `.php` | PHP | `grep -nE "require\|include" <file>` |
| `.cpp` `.h` `.c` | C/C++ | `grep -nE "^#include" <file>` |

---

## Workflow

### 1 — Detect the main language

```bash
find "<repo_path>" -type f \( -name "*.py" -o -name "*.js" -o -name "*.ts" \
  -o -name "*.rs" -o -name "*.java" -o -name "*.go" -o -name "*.rb" \) \
  -not -path "*/.venv/*" -not -path "*/node_modules/*" \
  -not -path "*/__pycache__/*" 2>/dev/null \
  | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -5
```

The language with the most files is the primary one. Multi-language repos are supported.

### 2 — Collect all source files

```bash
find "<repo_path>" -type f \( -name "*.py" -o -name "*.js" -o -name "*.ts" \
  -o -name "*.jsx" -o -name "*.tsx" -o -name "*.rs" -o -name "*.java" \
  -o -name "*.go" -o -name "*.rb" -o -name "*.php" \
  -o -name "*.cpp" -o -name "*.h" -o -name "*.c" \) \
  -not -path "*/.venv/*" -not -path "*/node_modules/*" \
  -not -path "*/__pycache__/*" -not -path "*/.git/*" \
  -not -path "*/dist/*" -not -path "*/build/*" \
  2>/dev/null | sort
```

### 3 — Extract internal dependencies for each file

For each source file:

1. Extract import lines using the language pattern
2. Normalize the module name:
   - Python: `from src.utils import X` → look for `utils.py`
   - JS/TS: `import X from './utils'` → look for `utils.js` or `utils.ts`
   - Remove relative prefixes (`./`, `../`, package name)
3. Search for the file in the repo:

```bash
find "<repo_path>" -name "<module_name>.*" \
  -not -path "*/.venv/*" -not -path "*/node_modules/*" \
  -not -path "*/__pycache__/*" 2>/dev/null | head -3
```

4. Found → internal dependency → record
5. Not found → external library → skip

### 4 — Write dependency-graph.md

```markdown
# Dependency Graph — <project_name>

Internal dependency map between project files.
Generated on <date> by scanning <N_files> source files.
Updated incrementally by daily-project-chronicle.

**Primary language**: <language>
**Files scanned**: <N>
**Internal dependencies found**: <N>

---

## Dependencies

| File | Depends on | Type | Updated |
|---|---|---|---|
| <filename> | <dependency_name> | imports | <date> |

---

## High-impact files (hotspots)

Files with the most inbound dependencies — changes here have the greatest cascade impact.

| File | Inbound dependencies | Notes |
|---|---|---|
| <filename> | <N> | |
```

Populate the hotspots table by counting how many times each file appears in the "Depends on" column, sorted descending.

### Absolute rules

- Never record external libraries
- Never record files from `.venv/`, `node_modules/`, `__pycache__/`
- If a file has no internal dependencies, do not add it to the table
- Use `Write` — this is a creation from scratch
- If `dependency-graph.md` already exists, warn and ask for confirmation before overwriting
