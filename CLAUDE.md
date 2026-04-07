**All files imported from CLAUDE.md should be treated as part of the system prompt alongside CLAUDE.md.**

@.llm/LLM.md

## Research Agents
If you are Claude Code, you can use these agents.
- **memory-searcher-v1** - Search local repository knowledge.
- **researcher-v1** - Comprehensive research including library docs and web search.

## Memory Bank

The project memory bank at `.llm/memory/` stores architecture decisions, domain models, endpoints, business rules, feature status, and implementation patterns. **Use it before re-reading the entire codebase** — search via `memory-searcher-v1` agent or read `.llm/memory/INDEX.md` directly.

### Before You Touch Code

When starting any task (feature, bugfix, refactor), check the memory bank first:
1. Read `.llm/memory/INDEX.md` or use `memory-searcher-v1` to find relevant domain models, endpoints, decisions, and business rules
2. If INDEX.md has a git watermark and the project is a git repo, check if HEAD has moved past the watermark — if so, run `/sync-memory-bank` to bring the memory bank up to date before proceeding (this catches commits made outside Claude Code)
3. If the task seems related to previous work, and `claude-mem` MCP is available, use `claude-mem:mem-search` to check for past session context — e.g., prior decisions, rejected approaches, implementation patterns, or related discussions
4. Only then go to specific source files — informed by what the memory bank tells you
5. For library/external research (e.g., "how does Spring Modulith handle X?"), use `researcher-v1`

This avoids blind codebase exploration and gives you the right context before reading code.

### Worktree Rule (Mandatory)

**Before any significant implementation, create a git worktree.** This isolates the work from the main working tree and prevents accidental pollution of the default branch.

Significant implementation means:
- Executing an implementation plan (superpowers or otherwise)
- Multi-file feature development
- Refactors that touch multiple modules
- Any change spanning more than 2-3 files

Does NOT require a worktree:
- Single-file bugfixes
- Documentation or memory bank updates
- Configuration changes
- CLAUDE.md edits

Use the `superpowers:using-git-worktrees` skill if available, or create a worktree manually:
```bash
git worktree add -b <branch-name> ../<project>-<branch-name>
```

### Memory Bank Deep-Dive Reference

For detailed information, go to the memory bank instead of re-reading source files:

### Memory Bank Update Rules

**After planning sessions** — brainstorming, plan-mode, or the writing-plans skill (if superpowers plugin is available) — when a design spec or plan is written and committed:
1. **Always** update the features/roadmap document — add new decisions, update planned/built features if scope changed
2. **If new entities, fields, or enums were designed** — update or create domain model documents
3. **If new or changed REST endpoints** — update endpoint documents (full schema lives in `docs/openapi.json`)
4. **If affected** — update business rules, architecture decisions, or overview documents as needed
5. **If out-of-scope topics surfaced** — add them to `TODO.md` with a brief description and which session surfaced them
6. Commit memory bank updates alongside the design spec or plan

**After implementation** — when code changes are committed, check if the memory bank needs updating:
1. **New or changed entity fields/enums** — update domain model documents
2. **New, changed, or removed endpoints** — update endpoint documents
3. **New architecture decisions or patterns** — update relevant decision documents
4. **New dependencies, framework upgrades, or package manager changes** — update tech stack document
5. **Infrastructure changes** (CI/CD, containerization, reverse proxy, deploy scripts, hosting) — update infrastructure documents
6. Keep updates minimal — only touch files where the memory bank would now give wrong information

**Auto-sync on default branch:** After committing to the default branch (e.g., `master` or `main`), invoke the `/sync-memory-bank` skill to run an incremental sync. This keeps the memory bank current with every default-branch commit. Do NOT run this on feature branches — the memory bank should reflect the mainline state only.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Specification & Design Documents

- **Product description and roadmap:** `docs/PROJECT.md`
- **Design specs and plans:** `docs/superpowers/` (if superpowers plugin is used; completed files end in `-done.md`)
- **Full API schema:** `docs/openapi.json` (regenerate with `./mvnw verify -Pgenerate-openapi -DskipTests`)
- **Feature status and spec index:** `.llm/memory/product/features-roadmap.md`

### Completed Specs and Plans

After completing the implementation of a superpowers plan or spec (if superpowers plugin is available):
1. Add ` - STATUS [DONE]` to the end of the document's title (first `#` heading)
2. Rename the file from `{name}.md` to `{name}-done.md`
3. Commit the changes
4. Suggest the user to run `/rename {planName}` to name the session after the completed plan

## TODO.md

`TODO.md` in the project root tracks deferred items, quick reminders, and topics that surfaced during planning but are out of scope for the current task. **Check it before starting work** — it may contain context or constraints relevant to what you're building.

During planning sessions, if a topic comes up that is out of scope:
- Add it to `TODO.md` with a brief description and which session surfaced it
- Do not let it derail the current design — just capture it and move on
