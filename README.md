# Claude Memory Bank

A persistent memory bank system for [Claude Code](https://claude.ai/code). Gives Claude long-term memory across sessions by storing architecture decisions, domain models, endpoints, business rules, and implementation patterns in a structured, searchable knowledge base committed alongside your code.

## What You Get

- **Memory bank** (`.llm/memory/`) — structured knowledge base with sections for domains, endpoints, decisions, overviews, techniques, product, and infrastructure
- **`sync-memory-bank` agent** — automatically builds or updates the memory bank from your codebase (full sync for new projects, incremental sync for ongoing work)
- **`/find-best-practices` skill** — searches GitHub and the web for community-vetted Claude Code skills for your language and framework, compares candidates, and installs the best one
- **`memory-searcher-v1` agent** — specialized sub-agent for fast memory bank lookups
- **`researcher-v1` agent** — comprehensive research agent combining library docs (Context7), memory bank, web search, and local codebase search
- **Scratchpad system** (`.llm/scratchpad/`) — temporary storage for preserving context within conversations
- **Instruction files** — runtime guidelines that teach Claude how to use and maintain the memory bank

## Installation

In Claude Code, say:

> Setup github.com/andriiblyzniuk/claude-memory-bank

Claude will read these instructions and set everything up automatically.

## Setup Instructions (for Claude)

When a user asks you to set up this memory bank, follow these steps exactly:

### Step 1: Download the install directory

Download the contents of the `install/` directory from this repository and place them in the user's project root. The `install/` directory mirrors the target project structure:

- `install/.llm/` → project root `.llm/`
- `install/.claude/` → project root `.claude/`

Use this command to download and extract:

```bash
curl -L https://github.com/krasav4k1/claude-memory-bank/archive/main.tar.gz \
  | tar -xz --strip-components=1 -C . claude-memory-bank-main/install/
```

This places `.llm/` and `.claude/` directly in the project root.

### Step 2: Handle CLAUDE.md

Check if `CLAUDE.md` exists in the project root:

- **If it does NOT exist:** Move copy install/CLAUDE.md into the Root directory of the project

- **If it DOES exist:** Update the existing project CLAUDE.md file with the content from install/CLAUDE.md. **PREPEND to the existing CLAUDE.md**

### Step 3: Initialize the memory bank

Run `@"sync-memory-bank (agent)"` to populate the memory bank from the existing codebase. For greenfield projects with no code yet, skip this step — the memory bank will be populated as the project grows.

### Step 4: Commit

Stage and commit all changes:

```bash
git add .llm/ .claude/ CLAUDE.md
git commit -m "feat: add claude memory bank system"
```

## Usage

### Memory Bank

Claude automatically checks `.llm/memory/INDEX.md` before exploring the codebase. The memory bank gets updated after planning sessions and implementation work.

To manually sync the memory bank after making changes outside of Claude Code:

> @"sync-memory-bank (agent)"

### Finding Best Practices

To discover and install community-vetted Claude Code skills for your tech stack:

> /find-best-practices

Or specify a technology directly:

> /find-best-practices react

### Directory Structure

After installation, your project will have:

```
your-project/
├── CLAUDE.md                          # Contains @.llm/LLM.md import
├── .llm/
│   ├── .gitignore                     # Excludes scratchpad/, *.local.*
│   ├── LLM.md                         # Entry point for all memory bank instructions
│   ├── instructions/
│   │   ├── runtime/
│   │   │   ├── scratchpad.md          # Temporary storage guidelines
│   │   │   ├── memory-bank.md         # Memory bank structure & format rules
│   │   │   └── memory-bank-searching.md  # Search instructions for agent
│   │   └── mcp/
│   │       └── context7.md            # Context7 MCP integration (optional)
│   ├── memory/
│   │   ├── INDEX.md                   # Master index of all memory documents
│   │   ├── decisions/                 # Architecture choices
│   │   ├── domains/                   # Domain models, entities, enums
│   │   ├── endpoints/                 # API reference
│   │   ├── infrastructure/            # Deployment, CI/CD, hosting
│   │   ├── overviews/                 # Big-picture architecture
│   │   ├── product/                   # Business rules, features, roadmap
│   │   └── techniques/                # Complex cross-module flows
│   └── scratchpad/                    # Temporary storage (gitignored)
└── .claude/
    ├── settings.local.json            # Permissions for skills
    ├── skills/
    │   └── find-best-practices/SKILL.md  # Best practices finder
    └── agents/
        ├── memory-searcher-v1.md      # Memory bank search agent
        ├── sync-memory-bank.md        # Memory bank sync skill agent
        └── researcher-v1.md           # Comprehensive research agent
```

## License

MIT
