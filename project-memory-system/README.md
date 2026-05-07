# Project Memory System

A persistent memory ecosystem for software projects, inspired by Karpathy's LLM Wiki pattern. Gives Claude complete, always-current context about any project — answering questions about history, decisions, dependencies, and current state.

## Core principle

Memory should be synthesis, not retrieval. Chronicles compile knowledge into structured indexes so Claude always starts from a pre-synthesized state instead of re-reading the entire history from scratch on every question.

## Repository structure

```
project-memory-system/
├── ONBOARDING.md                          ← start here
├── README.md                              ← this file
├── skills/
│   ├── README.md
│   ├── daily-project-chronicle/           ← generates chronicle + downstream agent pipeline
│   │   ├── SKILL.md
│   │   └── agents/
│   │       ├── chronicler.md              ← writes the daily note
│   │       ├── decisions-agent.md         ← updates decisions-index
│   │       ├── module-registry-agent.md   ← updates module-registry
│   │       ├── dependency-graph-agent.md  ← updates dependency-graph
│   │       └── snapshot-agent.md          ← updates snapshot (structural changes only)
│   ├── read-chronicles/
│   │   └── SKILL.md                       ← query the vault in natural language
│   ├── manage-issues/
│   │   ├── SKILL.md
│   │   └── agents/
│   │       └── issue-manager.md
│   ├── project-snapshot/                  ← one-shot snapshot generation
│   │   ├── SKILL.md
│   │   └── agents/
│   │       └── snapshot-builder.md
│   ├── init-dependency-graph/             ← one-shot full repo scan
│   │   ├── SKILL.md
│   │   └── agents/
│   │       └── dependency-scanner.md
│   ├── commit-semantic-extract/           ← on-demand semantic commit extraction
│   │   ├── SKILL.md
│   │   └── agents/
│   │       └── semantic-extractor.md
│   └── vault-lint/                        ← periodic vault health check
│       ├── SKILL.md
│       └── agents/
│           └── lint-agent.md
└── vault-template/
    ├── project.config.yaml.example
    ├── .gitignore
    ├── Daily/                             ← generated daily chronicles
    ├── Project/
    │   ├── snapshot.md                    ← current project state
    │   ├── decisions-index.md             ← all architectural decisions
    │   ├── module-registry.md             ← all key modules and files
    │   ├── open-issues.md                 ← open, in-progress, resolved issues
    │   └── dependency-graph.md            ← internal file dependency map
    ├── Meta/
    │   └── vault-index.md                 ← vault navigation map
    └── Templates/
        └── daily-chronicle-template.md
```

## Skills overview

| Skill | Purpose | Frequency |
|---|---|---|
| `daily-project-chronicle` | End-of-day note + downstream pipeline | Daily |
| `read-chronicles` | Natural language queries over the vault | On-demand |
| `manage-issues` | Track bugs and open problems | On-demand |
| `project-snapshot` | Generate initial snapshot from existing chronicles | One-shot |
| `init-dependency-graph` | Full repo dependency scan | One-shot |
| `commit-semantic-extract` | Semantic queries over commit history | On-demand |
| `vault-lint` | Vault health check — fix broken links, flag contradictions | Monthly |

## Daily chronicle agent pipeline

After writing the note, Claude evaluates the structural diff and conditionally activates downstream agents:

```
Orchestrator
│
├── Chronicler agent (always runs)
│
└── Claude evaluates structural diff:
    ├── New decisions?          → Decisions agent
    ├── New modules?            → Module Registry agent
    ├── Modified files?         → Dependency Graph agent
    └── Structural changes?     → Snapshot agent
```

Structural triggers for the Snapshot agent: files added, renamed or deleted; folders created or moved; output format changes; external dependencies added or removed.

## Compatibility

- **Vault**: Obsidian (recommended) or any Markdown editor
- **Repository**: any git repository
- **Languages**: Python, JavaScript, TypeScript, Rust, Java, Go, Ruby, PHP, C/C++
- **OS**: Linux, macOS, Windows (with WSL or Cowork)
- **Claude**: claude.ai with custom skills support (Cowork or equivalent)

## Contributing

Pull requests welcome. When modifying skills, always test on a sample vault before proposing changes. `project.config.yaml` is gitignored because it contains machine-specific absolute paths — do not commit it.
