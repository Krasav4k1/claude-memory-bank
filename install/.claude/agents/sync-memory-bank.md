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

**Structure & Stack:**
- List top-level directories, identify module/package layout
- Find entry points (main class, app bootstrap)
- Identify language, version, framework, build tool, package manager from config files
- Extract key dependencies and versions from the manifest

**Architecture & Patterns:**
- Identify module boundaries and enforcement mechanism
- Map cross-module communication patterns (events, direct calls, shared interfaces)
- Identify recurring patterns and abstractions
- Document shared infrastructure (auth, error handling, logging, filters)

**Domain & Business Logic:**
- Find entity/model classes — extract fields, types, enums, relationships, key business methods
- Identify status lifecycles and state machines
- Document validation rules, business constraints, domain invariants
- Map data flow from API request through service layer to database

**Endpoints & API (if applicable):**
- Find controllers/handlers — extract routes, HTTP methods, auth requirements
- Identify request/response DTOs
- Document public vs authenticated endpoints

**Decisions & Conventions:**
- Look for deliberate architectural choices
- Document API conventions (error format, headers, versioning)
- Identify auth strategy and permission model
- Understand testing approach and test configuration

### 3. Create INDEX.md and Write Documents

If `.llm/memory/INDEX.md` doesn't exist, create it. Define sections based on what the codebase actually contains — read `.llm/instructions/runtime/memory-bank.md` for guidance on section types.

Each INDEX.md section needs a `## Section Name` header, a description line, and document entries with one-line summaries.

For each document:
1. Create the directory if needed (`mkdir -p .llm/memory/<section>/`)
2. Write with YAML frontmatter (`title`, `tags`)
3. Keep concise — 50-100 lines
4. Include `## Relevant Code` with file paths
5. Include `## Further Reading` with cross-references
6. Add to INDEX.md with a one-line summary

**Writing priorities:** Architecture overview first, then domain models, tech stack, decisions, endpoints, business rules, techniques.

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
