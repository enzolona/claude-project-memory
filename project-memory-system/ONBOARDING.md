# Onboarding — Project Memory System

Welcome. This guide takes you from zero to a working memory system in under 15 minutes, whether you're starting a new project or adding memory to an existing one.

---

## What this system does

Project Memory System is a persistent memory layer for software projects that runs on top of Claude. Every evening you write a note (the **chronicle**), and the system automatically builds structured memory that Claude can query in natural language.

**What you can ask Claude after installing:**

```
"What did we work on this week?"
"Why did we choose that library?"
"When did module X break?"
"What issues are still open?"
"Show me the evolution of this file over the last 30 days."
"What contributed to the authentication feature?"
```

Claude answers by reading from compiled indexes — not by re-scanning every chronicle from scratch each time.

---

## Prerequisites

Before you begin, make sure you have:

- **Claude** with custom skill support (Cowork or equivalent)
- **A git repository** — any language, any size
- **Obsidian** (recommended) or any Markdown editor for the vault
- **Git** installed and accessible from the terminal

---

## Installation

### Step 1 — Download the repository

```bash
git clone https://github.com/<your-username>/project-memory-system.git
cd project-memory-system
```

Or download the ZIP from the GitHub page and extract it.

---

### Step 2 — Create your vault

The vault is the folder where all memory files for your project are stored. It can live anywhere on your filesystem — it does not need to be inside your project repo.

```bash
# Choose any path you prefer
mkdir -p ~/Documents/my-project-vault
cp -r vault-template/* ~/Documents/my-project-vault/
```

Rename the config file:

```bash
cp ~/Documents/my-project-vault/project.config.yaml.example \
   ~/Documents/my-project-vault/project.config.yaml
```

Open it and fill in your values:

```yaml
project_name: my-project          # your project name
repo_path: /path/to/your/repo     # absolute path to the git repository
vault_path: /path/to/your/vault   # absolute path to the vault you just created
language: english                  # english | italiano
```

> **Note**: `project.config.yaml` is in the vault's `.gitignore` because it
> contains machine-specific absolute paths. Do not commit it if you share the
> vault on git.

---

### Step 3 — Install the skills

Copy the `skills/` folder to the directory Claude uses for custom skills.

**On Cowork**, the typical path is `/mnt/skills/user/`. Copy each skill subfolder:

```bash
cp -r skills/daily-project-chronicle /mnt/skills/user/
cp -r skills/read-chronicles         /mnt/skills/user/
cp -r skills/manage-issues           /mnt/skills/user/
cp -r skills/project-snapshot        /mnt/skills/user/
cp -r skills/init-dependency-graph   /mnt/skills/user/
cp -r skills/commit-semantic-extract /mnt/skills/user/
cp -r skills/vault-lint              /mnt/skills/user/
```

Verify Claude can see them by asking: `"what skills do you have?"`.

---

### Step 4 — First run

**New project** (no existing history):

Open Claude and type:

```
write today's chronicle
```

Claude will ask for `vault_path`, `repo_path`, `project_name`, and `language` once, then create `project.config.yaml` automatically. From this point on it reads the config silently.

---

**Existing project** (already has code and history):

Run these three commands in order:

```
1. "initialize the dependency graph"
```
Claude scans the entire repo and builds `dependency-graph.md`. Takes 1–3 minutes on large repos.

```
2. "generate the project snapshot"
```
Claude reads all available chronicles (if any) and the repo structure to produce `snapshot.md` — the current state of the project.

```
3. "write today's chronicle"
```
From now on, the daily pipeline runs automatically.

---

## Daily workflow

The only thing you need to do every day:

```
write today's chronicle
```

Claude will:
1. Read `git diff`, commit messages, test logs, and modified source files
2. Write a dense structured note in `Daily/YYYY-MM-DD-daily-project-chronicle.md`
3. Evaluate the structural diff and decide which downstream agents to run
4. Update `decisions-index.md`, `module-registry.md`, `dependency-graph.md`, and/or `snapshot.md` as needed
5. Report what was written and which indexes were updated

---

## Querying the vault

Ask Claude anything about your project in natural language. No special syntax needed.

### Status queries
```
"What did we do this week?"
"What are the open items from the last session?"
"Summarize the last two weeks."
```

### Decision queries
```
"Why did we choose X over Y?"
"When did we decide to change the output format?"
"Show me all architectural decisions."
```

### Module and file queries
```
"When was module X introduced?"
"Show me the evolution of pipeline.py."
"What files depend on detector.py?"
```

### Error and fix queries
```
"When did the tests start failing?"
"Find this error in the chronicle: AssertionError in test_loader"
"When was this bug fixed?"
```

### Issue queries
```
"What issues are still open?"
"Show me all high-priority issues."
"When was ISSUE-003 resolved?"
```

---

## Managing issues

```
"add issue: description of the problem"       → creates ISSUE-001, ISSUE-002, ...
"mark ISSUE-003 as resolved"                  → moves it to Resolved with today's date
"list open issues"                             → shows the Open table
"list all issues"                             → shows all three tables
"details on ISSUE-005"                        → full detail of that issue
```

---

## Periodic maintenance

Once a month, run:

```
"lint the vault"
```

Claude will:
- **Auto-fix** broken links, incorrect issue counters, orphan registry entries
- **Flag for your review** semantic contradictions between the snapshot, decisions, and actual repo structure

For each flag, Claude asks a direct question and waits for your answer before applying changes.

---

## Vault structure explained

| File | Purpose | Updated by |
|---|---|---|
| `Daily/YYYY-MM-DD-daily-project-chronicle.md` | Dense daily note | `daily-project-chronicle` |
| `Project/snapshot.md` | Current project state | `snapshot-agent` (structural changes) |
| `Project/decisions-index.md` | All architectural decisions | `decisions-agent` |
| `Project/module-registry.md` | All key modules and files | `module-registry-agent` |
| `Project/open-issues.md` | Issues with status and links | `manage-issues` |
| `Project/dependency-graph.md` | Internal file dependency map | `dependency-graph-agent` |
| `Meta/vault-index.md` | Navigation map | Manual |
| `Meta/lint-YYYY-MM-DD.md` | Lint reports | `vault-lint` |

---

## Using the vault across multiple machines

The vault files (except `project.config.yaml`) are plain Markdown — they can be versioned on git or synced with Obsidian Sync.

**Recommended setup for multi-machine use:**

1. Create a git repo for the vault: `git init ~/Documents/my-project-vault`
2. The `.gitignore` already excludes `project.config.yaml` and Obsidian workspace files
3. On each machine, create a local `project.config.yaml` with machine-specific paths
4. Commit and push the `Daily/`, `Project/`, `Meta/`, and `Templates/` folders normally

Skills are installed per-machine and do not need to be versioned.

---

## Troubleshooting

**Claude doesn't find the vault:**
Make sure `project.config.yaml` exists in your vault root and the paths are absolute (not relative).

**Chronicle is empty or minimal:**
Check that the `repo_path` in config points to a valid git repository with at least one commit. Run `git log` manually to verify.

**Downstream agents don't run:**
The orchestrator evaluates the structural diff after the chronicle is written. If no files were modified today, most agents correctly skip. This is expected behavior.

**Dependency graph is incomplete:**
Run `init-dependency-graph` to do a full scan. The incremental updates only cover files modified on the current day.

**Snapshot is outdated:**
The snapshot only updates on structural changes. If you want to force a full regeneration, run `generate the project snapshot` — it will overwrite the existing file.

---

## Getting help

If Claude gives unexpected results, try being more specific:

```
# Instead of:
"what happened?"

# Try:
"what did we work on between May 1 and May 7 in the pipeline module?"
```

For bugs or improvement ideas, open an issue on the GitHub repository.
