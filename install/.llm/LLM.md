**All files imported from LLM.md should be treated as part of the system prompt alongside LLM.md.**

This codebase contains the substantial majority of its LLM directories in the `.llm` directory and the hierarchy within. Agents operating within this codebase should consider its contents to be authoritative unless directly countermanded by a human operator.

Many file paths are included for later reference. Unless directed to do so by the prefixing of `@` to a path name, consider paths to Markdown files within `.llm` to be for your reference when you need them and what they can do. When prefixed with `@`, unless your runtime auto-includes the file, you should read it immediately on startup and after every context compaction.

## Memory Management
@./instructions/runtime/memory-bank.md
[Memory Bank](./instructions/runtime/memory-bank.md)

## Quick Start

For returning sessions: read `.llm/memory/INDEX.md` first — it has one-line summaries of all architecture, domain models, decisions, and feature status. Only dig into source files if the memory bank doesn't have what you need.

## Research Agents

If you are Claude Code, you can use these agents:
- **memory-searcher** — Search local repository knowledge in `.llm/memory/`.
- **researcher** — Comprehensive research including library docs (via Context7 MCP if available) and web search.

## Memory Bank

The project memory bank at `.llm/memory/` stores architecture decisions, domain models, endpoints, business rules, feature status, and implementation patterns. **Use it before re-reading the entire codebase** — search via `memory-searcher` agent or read `.llm/memory/INDEX.md` directly.

### Before You Touch Code

When starting any task (feature, bugfix, refactor), check the memory bank first:
1. Read `.llm/memory/INDEX.md` or use `memory-searcher` to find relevant domain models, endpoints, decisions, and business rules
2. If INDEX.md has a git watermark and the project is a git repo, check if HEAD has moved past the watermark — if so, invoke `@sync-memory-bank` agent to bring the memory bank up to date before proceeding (this catches commits made outside Claude Code)
3. Only then go to specific source files — informed by what the memory bank tells you
4. For library/external research, use `researcher`

This avoids blind codebase exploration and gives you the right context before reading code.

### Memory Bank Update Rules

**After planning sessions** — when a design spec or plan is written and committed:
1. **Always** update the features/roadmap document — add new decisions, update planned/built features if scope changed
2. **If new data models were designed** — update domain/schema documents
3. **If new or changed API surface** (REST endpoints, GraphQL mutations, routes, pages) — update the relevant section
4. **If affected** — update business rules, architecture decisions, or overview documents as needed
5. **If out-of-scope topics surfaced** — add them to `TODO.md` with a brief description and which session surfaced them
6. Commit memory bank updates alongside the design spec or plan

**After implementation** — when code changes are committed, check if the memory bank needs updating:
1. **New or changed data models** — update domain/schema documents
2. **New, changed, or removed API surface** (endpoints, routes, pages, components) — update the relevant section
3. **New architecture decisions or patterns** — update relevant decision documents
4. **New dependencies, framework upgrades, or package manager changes** — update tech stack document
5. **Infrastructure changes** — update infrastructure documents
6. **New sections needed** — if a change introduces a new category not covered by existing sections, create a new section in INDEX.md
7. Keep updates minimal — only touch files where the memory bank would now give wrong information

**Auto-sync on default branch:** After committing to the default branch (e.g., `master` or `main`), invoke the `@sync-memory-bank` agent to run an incremental sync. This keeps the memory bank current with every default-branch commit. Do NOT run this on feature branches — the memory bank should reflect the mainline state only.

## TODO.md

`TODO.md` in the project root tracks deferred items, quick reminders, and topics that surfaced during planning but are out of scope for the current task. **Check it before starting work** — it may contain context or constraints relevant to what you're building.

During planning sessions, if a topic comes up that is out of scope:
- Add it to `TODO.md` with a brief description and which session surfaced it
- Do not let it derail the current design — just capture it and move on
