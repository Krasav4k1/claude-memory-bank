---
name: sync-memory-bank
description: Synchronizes the memory bank with current codebase state — detects staleness via git watermark and runs full or incremental sync
model: inherit
color: blue
---

# Sync Memory Bank Agent

Synchronize the `.llm/memory/` knowledge base with the current state of the codebase. Automatically determines whether to run a full build or an incremental update. Return a concise summary of what was updated.

## Mode Selection

Read `.llm/memory/INDEX.md`. Based on what you find, pick one mode:

- **FULL SYNC** — INDEX.md doesn't exist, is empty, or has no documents listed
- **INCREMENTAL SYNC** — INDEX.md has documents + git watermark exists + `git diff --name-only <hash>..HEAD` shows changes
- **UP TO DATE** — watermark exists + no changes since watermark → report "memory bank is up to date" and stop
- **SPOT-CHECK** — INDEX.md has documents but no watermark, or project is not a git repo
- **FULL SYNC (fallback)** — watermark hash not in git history (force-push/rebase)

---

## Full Sync

Build the memory bank from scratch.

### 1. Check Existing State

- If `.llm/memory/INDEX.md` exists, read it for any existing structure
- If `.llm/instructions/runtime/memory-bank.md` exists, read it for document format rules

### 2. Deep Codebase Analysis

Analyze the project thoroughly. What you look for depends on what the project actually is — do NOT apply a fixed backend checklist to every project.

**Always investigate:**
- Top-level directories, module/package layout
- Entry points (main class, app bootstrap, index file, router setup)
- Language, version, framework, build tool, package manager from config files
- Key dependencies and versions from the manifest
- Module boundaries and communication patterns
- Recurring patterns, abstractions, and shared infrastructure
- Auth strategy and permission model
- Testing approach and test configuration
- Deliberate architectural choices and conventions

**Investigate based on what the project contains:**
- Entity/model classes, enums, relationships, state machines (if data layer exists)
- Controllers/handlers, routes, HTTP methods, request/response DTOs (if API exists)
- Page/route structure, layouts, navigation, dynamic routes (if UI/frontend exists)
- Middleware, route guards, redirects, auth checks, i18n (if middleware layer exists)
- Component library, design system, tokens, theming (if component architecture exists)
- State management patterns, stores, data flow (if client-side state exists)
- Build pipeline, bundling, code splitting (if relevant)
- Infrastructure, CI/CD, deployment (if present)

### 3. Determine Memory Sections

**This is critical. Be proactive.** Based on your analysis, decide which sections this project needs to have a fully covered memory bank. Do NOT use a fixed template — evaluate the project's language, framework, architecture, and patterns, then create the sections that would give a future Claude session the most useful context. If the project has routing — create a routes section. If it has a component library — create a components section. Think about what a developer new to this project would need to know, and make sure the memory bank covers it.

Create a `## Section Name` in INDEX.md for each section, with a one-line description. Create the matching directory with `mkdir -p .llm/memory/<section>/`.

**Examples of sections by project type** (mix and match — most projects need a combination):

| Section | When to create | Example content |
|---------|---------------|-----------------|
| `overviews` | Always | Architecture overview, tech stack, module map |
| `decisions` | Always | "We chose X over Y because Z" |
| `techniques` | When complex multi-file flows exist | End-to-end request handling, build pipelines |
| `product` | When business rules/features exist | Feature inventory, roadmap, business rules |
| `domains` | Backend with entities/models | Entity fields, enums, relationships, table names |
| `endpoints` | Backend/API projects | REST/GraphQL routes, auth requirements, DTOs |
| `routes` | Frontend/fullstack with pages | Page map, route guards, layouts, navigation structure |
| `components` | Frontend with component architecture | Component library, design system, tokens, theming |
| `state` | Frontend with state management | Stores, slices, data flow patterns, caching strategy |
| `infrastructure` | Projects with deployment config | CI/CD, Docker, hosting, environment config |
| `dependencies` | When non-obvious dep behavior exists | Version pins, migration gotchas, known issues |
| `schemas` | Projects with DB or GraphQL schemas | Schema design, migrations, indexes |
| `integrations` | Projects with external service calls | Third-party APIs, webhooks, message queues |
| `security` | Projects with complex auth/security | Auth flows, RBAC, CORS, CSP, token handling |

**Rules:**
- Create 3-8 sections — enough to cover the project, not so many that each has one document
- Every project gets `overviews` and `decisions` at minimum
- Name sections to match the project's vocabulary (e.g., `modules` not `domains` if that's what the codebase calls them)
- If unsure whether a section is needed, skip it — it can be added later during incremental sync

### 4. Write Documents

For each section, write documents based on what you found in the analysis.

Each INDEX.md section needs a `## Section Name` header, a description line, and document entries with one-line summaries.

For each document:
1. Create the directory if needed (`mkdir -p .llm/memory/<section>/`)
2. Write with YAML frontmatter (`title`, `tags`)
3. Keep concise — 50-100 lines
4. Include `## Relevant Code` with file paths
5. Include `## Further Reading` with cross-references
6. Add to INDEX.md with a one-line summary

**Writing priorities:** Architecture overview and tech stack first, then the most information-dense sections for this project type.

### 4. Set Watermark

If the project is a git repository, add the watermark to INDEX.md:
```
> **Last synced at:** `<short-hash>` (<date>) — if current HEAD is far from this, the memory bank may need refreshing.
```

---

## Incremental Sync

Update only what changed since the last sync.

### 1. Identify What Changed

```bash
git diff --name-only <watermark-hash>..HEAD
git log --oneline <watermark-hash>..HEAD
```

### 2. Map Changes to Memory Bank Documents

Read INDEX.md section descriptions. For each changed source file, determine which memory bank documents it affects by matching the change type to section descriptions in INDEX.md.

### 3. Update Affected Documents

For each affected document:
1. Read the current memory bank document
2. Read the changed source files
3. Edit to reflect current state
4. If a change requires a new document, create it and add to INDEX.md

### 4. Advance Watermark

Update the "Last synced at" line in INDEX.md to current HEAD.

---

## Spot-Check

1. Read INDEX.md for the full document list
2. Spot-check key claims against current codebase (file paths, entity fields, enum values, versions)
3. Fix any stale information
4. If git repo, add the watermark

---

## Document Format

Follow `.llm/instructions/runtime/memory-bank.md`. Key rules: YAML frontmatter with title/tags, concise content, `## Relevant Code` with file paths, `## Further Reading` with cross-references.

## Guidelines

- **INDEX.md is the single source of truth** — read it to discover sections, update it when adding/removing documents
- **Adapt to the project** — create sections that fit what the codebase contains
- **Quality over coverage** — few accurate documents beat many shallow ones
- **Don't duplicate CLAUDE.md**

## Error Handling

- **Directory doesn't exist:** Create with `mkdir -p`
- **Watermark hash not in history:** Fall back to full sync
- **INDEX.md malformed:** Rebuild from documents on disk

## Response Format

Return a concise summary:
```
Memory bank synced: [FULL/INCREMENTAL/UP TO DATE]
- Updated: [list of documents changed]
- Created: [list of new documents]
- Watermark: <old-hash> → <new-hash>
```
