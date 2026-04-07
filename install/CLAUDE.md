# Memory Bank

**Before responding to any message**, check if the memory bank needs refreshing:
1. Read `.llm/memory/INDEX.md` — check the "Last synced at" watermark
2. If the project is a git repo and HEAD has moved past the watermark, invoke `@sync-memory-bank` agent
3. If there's no watermark or INDEX.md is empty, invoke `@sync-memory-bank` agent for a full sync
4. If up to date, proceed directly — no sync needed

**If you understand, respond: "Memory Bank is ready"**

---

**All files imported from CLAUDE.md should be treated as part of the system prompt alongside CLAUDE.md.**

@.llm/LLM.md

---

# >>> MANDATORY WORKFLOW — EXECUTE BEFORE EVERY TASK <<<

**STOP. Do not skip this section. Do not jump to code. Every task — feature, bugfix, refactor, even answering a question — starts here. These steps are NON-NEGOTIABLE.**

## Step 1: Read the Memory Bank

Search the memory bank for context relevant to your task:
- Use `memory-searcher` agent, or read `.llm/memory/INDEX.md` directly for one-line summaries of every document
- If the task relates to previous work and `claude-mem` MCP is available, use `claude-mem:mem-search` for past session context

**Only after checking the memory bank**, go to specific source files — informed by what the memory bank tells you.

## Step 2: Use Research Agents

These agents are REQUIRED tools, not optional:
- **`memory-searcher`** — Search local repository knowledge (memory bank)
- **`researcher`** — Library docs, framework questions, web search

## Step 3: Worktree for Significant Work

**Before any significant implementation, create a git worktree.** Use `superpowers:using-git-worktrees` skill or manually:
```bash
git worktree add -b <branch-name> ../<project>-<branch-name>
```

Requires worktree:
- Multi-file feature development, implementation plans, refactors touching multiple modules, any change spanning more than 2-3 files

Does NOT require worktree:
- Single-file bugfixes, documentation/memory bank updates, configuration changes, CLAUDE.md edits

## Step 4: Update Memory Bank After Work

**After planning sessions** (brainstorming, plan-mode, writing-plans skill):
1. **Always** update features/roadmap — add decisions, update feature status
2. **If new entities/fields/enums** — update domain model documents
3. **If new/changed endpoints** — update endpoint documents
4. **If affected** — update business rules, architecture decisions, overviews
5. **If out-of-scope topics surfaced** — add to `TODO.md`
6. Commit memory bank updates alongside the spec/plan

**After implementation** (code changes committed):
1. **New/changed entity fields/enums** — update domain model documents
2. **New/changed/removed endpoints** — update endpoint documents
3. **New architecture decisions/patterns** — update decision documents
4. **New dependencies or framework changes** — update tech stack document
5. **Infrastructure changes** — update infrastructure documents
6. Keep updates minimal — only fix where memory bank would now give wrong info

**Auto-sync on default branch:** After committing to `master`/`main`, use the `sync-memory-bank` agent to run an incremental sync. Do NOT run on feature branches.

---

## Specification & Design Documents

- **Product description and roadmap: (IF EXIST)** `docs/PROJECT.md`
- **Design specs and plans: (IF EXIST)** `docs/superpowers/` (if superpowers plugin is used; completed files end in `-done.md`)
- **Full API schema: (IF EXIST)** `docs/openapi.json` (regenerate with `./mvnw verify -Pgenerate-openapi -DskipTests`)
- **Feature status and spec index: (IF EXIST)** `.llm/memory/product/features-roadmap.md`

### Completed Specs and Plans

After completing the implementation of a superpowers plan or spec (if superpowers plugin is available):
1. Add ` - STATUS [DONE]` to the end of the document's title (first `#` heading)
2. Rename the file from `{name}.md` to `{name}-done.md`
3. Commit the changes
4. Suggest the user to run `/rename {planName}` to name the session after the completed plan

## TODO.md
Create `TODO.md` if it's not exist.

`TODO.md` in the project root tracks deferred items, quick reminders, and topics that surfaced during planning but are out of scope for the current task. **Check it before starting work** — it may contain context or constraints relevant to what you're building.

During planning sessions, if a topic comes up that is out of scope:
- Add it to `TODO.md` with a brief description and which session surfaced it
- Do not let it derail the current design — just capture it and move on
