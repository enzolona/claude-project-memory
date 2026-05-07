# Vault Index — <project_name>

Navigation map for the vault. Updated manually or by the `daily-project-chronicle` skill.

## Structure

```
vault/
├── project.config.yaml          ← project configuration (machine-specific, gitignored)
├── Daily/                       ← daily chronicles (generated)
│   └── YYYY-MM-DD-daily-project-chronicle.md
├── Project/
│   ├── snapshot.md              ← current project state
│   ├── decisions-index.md       ← all architectural decisions
│   ├── module-registry.md       ← all key modules and files
│   ├── open-issues.md           ← open, in-progress, and resolved issues
│   └── dependency-graph.md      ← internal file dependency map
├── Meta/
│   ├── vault-index.md           ← this file
│   └── lint-YYYY-MM-DD.md       ← lint reports (generated monthly)
└── Templates/
    └── daily-chronicle-template.md
```

## How to use this vault with Claude

### Write today's chronicle
```
write today's chronicle
```

### Query project history
```
what did we work on this week?
when was module X introduced?
why did we choose Y?
show me the evolution of file.py
what happened between May 1 and May 7?
```

### Manage issues
```
add issue: description of the problem
list open issues
mark ISSUE-001 as resolved
```

### Maintenance
```
lint the vault
```

## Available chronicles

<!-- This section can be updated manually or left empty -->
<!-- Chronicles are navigable from the Daily/ folder -->
