# Skills — Installation Guide

## Prerequisites

- Claude with custom skill support (Cowork or equivalent)
- A `skills/` directory accessible from Claude's filesystem

## Skill structure

Each skill consists of:
- `SKILL.md` — the main file Claude reads to understand what to do
- `agents/<agent>.md` — specialized sub-agents spawned by the orchestrator

## How to install

### Option A — Cowork (recommended)

1. Copy the `skills/` folder to the directory mounted by Cowork (e.g. `/mnt/skills/user/`)
2. Each subfolder becomes an available skill
3. Optionally customize the trigger phrases in the `description` field of each `SKILL.md` frontmatter

```bash
cp -r skills/daily-project-chronicle /mnt/skills/user/
cp -r skills/read-chronicles         /mnt/skills/user/
cp -r skills/manage-issues           /mnt/skills/user/
cp -r skills/project-snapshot        /mnt/skills/user/
cp -r skills/init-dependency-graph   /mnt/skills/user/
cp -r skills/commit-semantic-extract /mnt/skills/user/
cp -r skills/vault-lint              /mnt/skills/user/
```

### Option B — Manual

Copy the skill folders to wherever Claude can access them, then reference the path explicitly in your prompt when needed.

## First-time configuration

The first time you use any skill, Claude will ask interactively for:

| Parameter | Description | Example |
|---|---|---|
| `vault_path` | Path to your Obsidian vault | `/Users/name/Documents/my-vault` |
| `repo_path` | Path to the git repository | `/Users/name/projects/my-repo` |
| `project_name` | Project name | `my-project` |
| `language` | Note language | `english` or `italiano` |

These are saved to `project.config.yaml` in the vault and reused in future sessions.

## Available skills

| Skill | Primary triggers |
|---|---|
| `daily-project-chronicle` | "write today's chronicle", "daily note", "end of day summary" |
| `read-chronicles` | "what did we do", "when was X introduced", "history of", "find this error" |
| `manage-issues` | "add issue", "mark as resolved", "list open issues", "close issue" |
| `project-snapshot` | "generate project snapshot", "create snapshot" |
| `init-dependency-graph` | "initialize the dependency graph", "scan dependencies" |
| `commit-semantic-extract` | "what contributed to feature X", "commits related to" |
| `vault-lint` | "lint the vault", "health check", "check vault" |

## Updating skills

To update, replace the `SKILL.md` and `agents/*.md` files with the new versions. The vault and config do not need to be modified.
