# Distributable Memory Bank Package — Design Spec

## Problem

The `claude-memory-bank` project contains a useful memory bank system (`.llm/` directory with instructions, memory structure, watermarking), custom agents, and skills — but it has no installation mechanism. It's currently a single-project template with Spring Boot-specific content baked in. The goal is to make it a universal, distributable package that any Claude Code user can install into any project with a single command.

## Target Audience

Any Claude Code user who wants a persistent memory bank system, regardless of tech stack.

## Installation Method

**Single method:** Tell Claude Code `Setup github.com/andriiblyzniuk/claude-memory-bank`. Claude reads the README and follows numbered installation steps automatically.

No manual curl/tar one-liner. The README serves as both human documentation and Claude's installation script.

## Repository Structure

```
claude-memory-bank/
├── README.md                              # Human docs + Claude install instructions
├── LICENSE                                # MIT
├── .gitignore
├── install/                               # Mirrors target project structure exactly
│   ├── .llm/
│   │   ├── .gitignore                     # Excludes scratchpad/, *.local.*
│   │   ├── LLM.md                         # Entry point — user's CLAUDE.md imports this
│   │   ├── instructions/
│   │   │   ├── runtime/
│   │   │   │   ├── scratchpad.md          # Temporary storage guidelines
│   │   │   │   ├── memory-bank.md         # Memory bank structure & format
│   │   │   │   └── memory-bank-searching.md  # Search instructions for agent
│   │   │   └── mcp/
│   │   │       └── context7.md            # Context7 MCP integration (optional)
│   │   └── memory/
│   │       ├── INDEX.md                   # Template — empty sections, no watermark
│   │       ├── .keep
│   │       ├── decisions/
│   │       ├── domains/
│   │       ├── endpoints/
│   │       ├── infrastructure/
│   │       ├── overviews/
│   │       ├── product/
│   │       └── techniques/
│   └── .claude/
│       ├── settings.local.json            # Minimal permissions for skills/agents
│       ├── skills/
│       │   ├── sync-memory-bank/
│       │   │   └── SKILL.md
│       │   └── find-best-practices/
│       │       └── SKILL.md
│       └── agents/
│           ├── memory-searcher-v1.md
│           └── researcher-v1.md
```

The `install/` directory is a direct overlay — its contents map 1:1 to where files land in the target project.

## Content Changes

### Removed from distribution

- **Spring Boot skills:** swagger-endpoint-documentation, spring-reactive-logging, http-test-files-with-endpoints — removed entirely
- **Root CLAUDE.md:** Not distributed as a standalone file. Its universal content (memory bank rules, agent references, update rules) moves into `LLM.md`
- **Project-specific references:** `docs/PROJECT.md`, `docs/superpowers/`, `docs/openapi.json` — removed from all instruction files
- **Superpowers-specific workflows:** "Completed Specs" renaming convention, worktree rule — removed (user may or may not have superpowers)

### Kept as-is

- **sync-memory-bank skill** — core functionality
- **find-best-practices skill** — useful universal utility
- **memory-searcher-v1 agent** — searches `.llm/memory/`
- **researcher-v1 agent** — comprehensive research with Context7, web, and local search
- **Context7 MCP instructions** — kept; users without Context7 can ignore it
- **Scratchpad system** — kept in instructions

### Modified for universal use

- **LLM.md** — becomes the single entry point. Contains:
  - Memory Bank usage rules (check before coding, search with agents)
  - Memory Bank Update Rules (after planning, after implementation)
  - Research Agents section (memory-searcher-v1, researcher-v1)
  - TODO.md reference (mentioned in instructions, not created during install)
  - `@` imports for runtime instructions (scratchpad, memory-bank)
  - Quick Start pointing to `.llm/memory/INDEX.md`
  - No Spring-specific content, no project-specific file paths

- **memory-bank.md** — remove references to `docs/openapi.json`, superpowers specs, and framework-specific examples. Keep generic: works for any project.

- **INDEX.md** — template version with empty section directories and one-line descriptions. No git watermark (set on first `/sync-memory-bank` run).

- **settings.local.json** — minimal permissions:
  - Read, Write, Edit, Glob, Grep (file operations for skills)
  - Bash: git commands only (for watermarking)
  - WebSearch, WebFetch (for find-best-practices and researcher)
  - Agent (for sub-agent dispatch)
  - Remove: mvnw, docker, npm, and all Spring-specific permissions

## Installation Flow (what Claude does)

When a user says `Setup github.com/andriiblyzniuk/claude-memory-bank`, Claude:

1. Fetches and reads the README from the repository
2. Downloads the `install/` directory contents
3. Copies `.llm/` to the project root
4. Copies `.claude/skills/` and `.claude/agents/` to the project root
5. Handles `.claude/settings.local.json`:
   - If none exists: copy as-is
   - If one exists: merge permissions from the distributed file into the existing one
6. Handles `CLAUDE.md`:
   - If none exists: create with `@.llm/LLM.md`
   - If one exists: append `@.llm/LLM.md` line
7. Runs `/sync-memory-bank` to populate the memory bank from the existing codebase
8. Commits the changes

## README Structure

1. **What is this** — one paragraph: persistent memory bank for Claude Code
2. **What you get** — bullet list of components (memory bank, skills, agents, scratchpad)
3. **Installation** — single command: tell Claude `Setup github.com/andriiblyzniuk/claude-memory-bank`
4. **Setup Instructions (for Claude)** — clearly numbered steps Claude follows
5. **Usage** — brief overview of `/sync-memory-bank`, `/find-best-practices`, agents
6. **Directory structure** — what gets installed and why

## What the User Experiences

1. Opens Claude Code in any project (new or existing)
2. Types `Setup github.com/andriiblyzniuk/claude-memory-bank`
3. Claude reads the README, copies files, updates CLAUDE.md, runs sync
4. Memory bank is live — Claude checks `.llm/memory/` before exploring code in all future sessions
