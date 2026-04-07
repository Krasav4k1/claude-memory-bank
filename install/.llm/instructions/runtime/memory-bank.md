The memory bank is a section of the repository (committed alongside other code) that serves as long-term memory across multiple sessions and multiple developers. It is the **first place to look** before re-reading source files — if the answer is in the memory bank, use it.

## Structure

The memory bank lives at `.llm/memory/`. Read `.llm/memory/INDEX.md` for the full directory structure, section descriptions, and document inventory. INDEX.md is the single source of truth for what exists in the memory bank — do not assume a fixed set of folders.

Subdirectories within each category are permitted when a topic grows large enough.

## Document Format

Documents are Markdown with YAML front matter:

```markdown
---
title: A Friendly Title
tags:
  - #authentication
  - #jwt
  - #oauth
---

Concise content here. Back-references to code files and other memory documents.

## Relevant Code

- `src/config/auth.ts`: configures authentication middleware
- `src/services/permission.ts`: maps roles to permissions
```

### Tag Rules
- Max 10 tags per document
- All tags start with `#`
- Use `_` instead of spaces: `#error_handling` not `#error handling`
- Good tags: library names, module names, major class names, subsystem identifiers

### Content Rules
- Keep documents concise — enough to answer "what decision was made and why" without reading source
- Technique documents may be longer since they explain end-to-end flows
- Always include `## Relevant Code` with file paths when referencing implementation
- Always include `## Further Reading` linking to related memory documents
- If the project uses design specs, reference them by path rather than duplicating content

## Relationship to Design Specs

If the project maintains design specs, the memory bank and specs serve different purposes:

| | Memory Bank | Design Specs |
|---|---|---|
| **Purpose** | Quick-reference knowledge base | Full design rationale and API contracts |
| **Audience** | Any future Claude session | Implementation sessions |
| **Granularity** | Summarized decisions and domain models | Complete endpoint definitions, request/response schemas, error matrices |
| **When to use** | First — to check if you already know the answer | When you need the full spec for a specific feature (e.g., exact validation rules, edge cases) |

**The memory bank should reference specs, not duplicate them.** When a memory document covers a topic that has a spec, include a link. The product/roadmap document in the memory bank tracks all completed and pending specs in one place.

## Finding Things

Use the `memory-searcher-v1` sub-agent for searches. It uses Grep and Glob within `.llm/memory/` only.

For quick lookups, read `INDEX.md` directly — it has one-line summaries of every document.

### Search Strategies
- **By tag:** Search frontmatter for tags like `#authentication`, `#billing`
- **By content:** Search document bodies for keywords, class names, error codes
- **By path pattern:** Use INDEX.md section headers to identify which directory to search
- **Combined:** Tag + content for precise results
- **For full spec details:** Check the product/roadmap document for the spec path, then read the spec directly

## Updating the Memory Bank

### When to Update

**After any planning session** — when a design spec or plan is written and committed:
1. **Always:** Update the features/roadmap document — add new decisions, update feature status
2. **If new entities, fields, or enums were designed:** Update or create domain model documents
3. **If new or changed REST endpoints:** Update endpoint documents
4. **If affected:** Update business rules, architecture decisions, or overview documents as needed
5. **If out-of-scope topics surfaced:** Add them to `TODO.md` (project root) with a brief description and which session surfaced them
6. Commit memory bank updates alongside the design spec or plan

**After implementation** — when code changes are committed, check if the memory bank needs updating:
1. **New or changed entity fields/enums** — update domain model documents
2. **New, changed, or removed endpoints** — update endpoint documents
3. **New architecture decisions or patterns** — update relevant decision documents
4. **New dependencies, framework upgrades, or package manager changes** — update tech stack document
5. **Infrastructure changes** (CI/CD pipelines, containerization, reverse proxy, deploy scripts, hosting, server provisioning) — update infrastructure documents
6. **Feature completed** — update the features/roadmap document (move from planned to built)
7. Keep updates minimal — only touch files where the memory bank would now give wrong information

**When you discover stale information:** If a memory document contradicts the current code:
1. Trust the code — code is always authoritative over memory
2. Update the memory document to match current state
3. If the change seems significant or surprising, flag it to the user before updating

### What NOT to Store
- Exact code snippets that will go stale — reference file paths instead
- Debugging sessions or temporary investigation notes — use scratchpad for those
- Information already in CLAUDE.md — don't duplicate
- Ephemeral task state — use tasks or scratchpad instead

### Creating New Documents
1. Check if an existing document covers the topic (search first)
2. If updating, edit in place — don't create duplicates
3. If creating new, check INDEX.md section descriptions to pick the right directory
4. Add an entry to `INDEX.md` with a one-line summary
5. Include cross-references to related documents

### Version Marker

If the project is a git repository, INDEX.md contains a "Last synced at" line with the commit hash and date. When making memory bank updates, update this marker to the current commit:
```
> **Last synced at:** `<short-hash>` (<date>) — if current HEAD is far from this, the memory bank may need refreshing.
```
This helps future sessions detect staleness — if HEAD has moved significantly past the marker, the memory bank may need review.

### Retiring Documents
- If a decision is reversed, update the document to reflect the new decision (don't delete — the history of "we tried X and switched to Y" is valuable)
- If a technique is obsolete (code deleted), archive by adding `[ARCHIVED]` to the title and a note explaining why
