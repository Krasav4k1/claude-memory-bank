# Claude Memory Bank

A persistent memory bank system for [Claude Code](https://claude.ai/code). Gives Claude long-term memory across sessions by storing architecture decisions, domain models, endpoints, business rules, and implementation patterns in a structured, searchable knowledge base committed alongside your code.

## What You Get

- **Memory bank** (`.llm/memory/`) — structured knowledge base with sections created dynamically based on your project type (e.g., routes/components for frontend, endpoints/domains for backend, or any mix)
- **`sync-memory-bank` agent** — automatically builds or updates the memory bank from your codebase (full sync for new projects, incremental sync for ongoing work)
- **`/find-best-practices` skill** — searches GitHub and the web for community-vetted Claude Code skills for your language and framework, compares candidates, and installs the best one
- **`memory-searcher` agent** — specialized sub-agent for fast memory bank lookups
- **`researcher` agent** — comprehensive research agent combining library docs (Context7), memory bank, web search, and local codebase search
- **Instruction files** — runtime guidelines that teach Claude how to use and maintain the memory bank

## Installation

In Claude Code, say:

> Setup github.com/Krasav4k1/claude-memory-bank

Claude will read these instructions and set everything up automatically.

## Setup Instructions (for Claude)

When a user asks you to set up this memory bank, follow these steps exactly:

### Step 1: Download to temp directory

Run this exact command — do NOT modify it:

```bash
curl -L https://github.com/krasav4k1/claude-memory-bank/archive/main.tar.gz \
  | tar -xz -C /tmp
```

This creates `/tmp/claude-memory-bank-main/` with the full repo. The files you need are in `/tmp/claude-memory-bank-main/install/`.

### Step 2: Copy to project root

Run these exact commands from the project root:

```bash
cp -r /tmp/claude-memory-bank-main/install/.llm .
cp -rn /tmp/claude-memory-bank-main/install/.claude .
```

If `.claude/` already exists, merge the contents:

```bash
cp -r /tmp/claude-memory-bank-main/install/.claude/agents .claude/
cp -r /tmp/claude-memory-bank-main/install/.claude/skills .claude/
```

### Step 3: Handle CLAUDE.md

**If `CLAUDE.md` does NOT exist** in the project root:
```bash
cp /tmp/claude-memory-bank-main/install/CLAUDE.md .
```

**If `CLAUDE.md` DOES exist**, prepend the memory bank content:
```bash
cat /tmp/claude-memory-bank-main/install/CLAUDE.md CLAUDE.md > CLAUDE.md.tmp && mv CLAUDE.md.tmp CLAUDE.md
```

### Step 4: Clean up

```bash
rm -rf /tmp/claude-memory-bank-main
```

### Step 5: Initialize the memory bank

Run `@"sync-memory-bank (agent)"` to populate the memory bank from the existing codebase. For greenfield projects with no code yet, skip this step — the memory bank will be populated as the project grows.

### Step 6: Commit

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
│   │   │   ├── memory-bank.md         # Memory bank structure & format rules
│   │   │   └── memory-bank-searching.md  # Search instructions for agent
│   ├── memory/
│   │   ├── INDEX.md                   # Master index — sections created dynamically per project
│   │   └── <sections>/               # Created by sync agent based on project type
└── .claude/
    ├── skills/
    │   └── find-best-practices/SKILL.md  # Best practices finder
    └── agents/
        ├── memory-searcher.md         # Memory bank search agent
        ├── sync-memory-bank.md        # Memory bank sync agent
        └── researcher.md              # Comprehensive research agent
```

## License

MIT
