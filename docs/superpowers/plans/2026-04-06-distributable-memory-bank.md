# Distributable Memory Bank Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure the claude-memory-bank repository into a distributable package that can be installed into any project via `Setup github.com/andriiblyzniuk/claude-memory-bank`.

**Architecture:** Create an `install/` directory mirroring the target project structure (`.llm/` + `.claude/`), genericize all content by removing Spring Boot and project-specific references, write a README that serves as both human documentation and Claude's installation script.

**Tech Stack:** Markdown, YAML frontmatter, JSON (settings), shell commands

---

### Task 1: Create the install/ directory structure

**Files:**
- Create: `install/.llm/.gitignore`
- Create: `install/.llm/memory/.keep`
- Create: `install/.llm/memory/decisions/.keep`
- Create: `install/.llm/memory/domains/.keep`
- Create: `install/.llm/memory/endpoints/.keep`
- Create: `install/.llm/memory/infrastructure/.keep`
- Create: `install/.llm/memory/overviews/.keep`
- Create: `install/.llm/memory/product/.keep`
- Create: `install/.llm/memory/techniques/.keep`

- [ ] **Step 1: Create all directories with .keep files**

```bash
cd /home/andriiblyzniuk/projects/claude-memory-bank
mkdir -p install/.llm/memory/{decisions,domains,endpoints,infrastructure,overviews,product,techniques}
mkdir -p install/.llm/instructions/runtime
mkdir -p install/.llm/instructions/mcp
mkdir -p install/.claude/skills/sync-memory-bank
mkdir -p install/.claude/skills/find-best-practices
mkdir -p install/.claude/agents
```

- [ ] **Step 2: Create .keep files for empty directories**

```bash
touch install/.llm/memory/.keep
touch install/.llm/memory/decisions/.keep
touch install/.llm/memory/domains/.keep
touch install/.llm/memory/endpoints/.keep
touch install/.llm/memory/infrastructure/.keep
touch install/.llm/memory/overviews/.keep
touch install/.llm/memory/product/.keep
touch install/.llm/memory/techniques/.keep
```

- [ ] **Step 3: Create install/.llm/.gitignore**

Write `install/.llm/.gitignore` with this content:

```
scratchpad/
!scratchpad/.keep

*.local.*
```

- [ ] **Step 4: Commit**

```bash
git add install/
git commit -m "scaffold: create install/ directory structure with empty memory bank sections"
```

---

### Task 2: Write the genericized INDEX.md template

**Files:**
- Create: `install/.llm/memory/INDEX.md`

The template INDEX.md should have no git watermark (set on first `/sync-memory-bank` run) and no project-specific content.

- [ ] **Step 1: Write INDEX.md**

Write `install/.llm/memory/INDEX.md` with this content:

```markdown
# Memory Bank Index

Quick-reference pointers into the memory bank. Read the file itself for details.

Each section below corresponds to a directory in `.llm/memory/`. When creating new documents, place them in the matching directory based on the section descriptions.

## Endpoints
REST API endpoint reference per module: HTTP method, path, auth requirement, request/response DTOs, purpose.


## Domains
Source of truth for domain models: entities with fields/types, enums with values, key business methods, table names, relationships. One file per module (or grouped for small modules). When you need to know what fields an entity has or what values an enum contains, look here.


## Overviews
Big-picture architecture only: module map, tech stack, cross-module communication patterns, schema-at-a-glance. Does NOT duplicate entity fields or enum values — references Domains section for those.


## Decisions
Architecture and implementation choices that constrain future work: "we chose X over Y because Z." Includes auth strategy, API conventions, testing approach, permission model.


## Techniques
Complex implementation flows that span multiple files or modules. How something actually works end-to-end.


## Product
Business rules, feature inventory, roadmap, planned vs built status. The "what" and "why" of the product.


## Infrastructure
Deployment, CI/CD, Docker, reverse proxy, server provisioning, and hosting configuration. How the application gets built, shipped, and run in production.


## Dependencies
Non-obvious dependency behavior, version pins, migration gotchas, known issues with third-party libraries. Created on demand when relevant.

_(No documents yet)_
```

- [ ] **Step 2: Commit**

```bash
git add install/.llm/memory/INDEX.md
git commit -m "scaffold: add template INDEX.md for memory bank"
```

---

### Task 3: Write the genericized LLM.md

**Files:**
- Create: `install/.llm/LLM.md`

This is the single entry point that the user's CLAUDE.md will import via `@.llm/LLM.md`. It must contain all the universal memory bank instructions previously split between CLAUDE.md and LLM.md, with no Spring Boot or project-specific references.

- [ ] **Step 1: Write LLM.md**

Write `install/.llm/LLM.md` with this content:

```markdown
**All files imported from LLM.md should be treated as part of the system prompt alongside LLM.md.**

This codebase contains the substantial majority of its LLM directories in the `.llm` directory and the hierarchy within. Agents operating within this codebase should consider its contents to be authoritative unless directly countermanded by a human operator.

Many file paths are included for later reference. Unless directed to do so by the prefixing of `@` to a path name, consider paths to Markdown files within `.llm` to be for your reference when you need them and what they can do. When prefixed with `@`, unless your runtime auto-includes the file, you should read it immediately on startup and after every context compaction.

## Runtime Instructions
@./instructions/runtime/scratchpad.md
[Scratchpad](./instructions/runtime/scratchpad.md)

## Memory Management
@./instructions/runtime/memory-bank.md
[Memory Bank](./instructions/runtime/memory-bank.md)

## Quick Start

For returning sessions: read `.llm/memory/INDEX.md` first — it has one-line summaries of all architecture, domain models, decisions, and feature status. Only dig into source files if the memory bank doesn't have what you need.

## Research Agents

If you are Claude Code, you can use these agents:
- **memory-searcher-v1** — Search local repository knowledge in `.llm/memory/`.
- **researcher-v1** — Comprehensive research including library docs (via Context7 MCP if available) and web search.

## Memory Bank

The project memory bank at `.llm/memory/` stores architecture decisions, domain models, endpoints, business rules, feature status, and implementation patterns. **Use it before re-reading the entire codebase** — search via `memory-searcher-v1` agent or read `.llm/memory/INDEX.md` directly.

### Before You Touch Code

When starting any task (feature, bugfix, refactor), check the memory bank first:
1. Read `.llm/memory/INDEX.md` or use `memory-searcher-v1` to find relevant domain models, endpoints, decisions, and business rules
2. If INDEX.md has a git watermark and the project is a git repo, check if HEAD has moved past the watermark — if so, run `/sync-memory-bank` to bring the memory bank up to date before proceeding (this catches commits made outside Claude Code)
3. Only then go to specific source files — informed by what the memory bank tells you
4. For library/external research, use `researcher-v1`

This avoids blind codebase exploration and gives you the right context before reading code.

### Memory Bank Update Rules

**After planning sessions** — when a design spec or plan is written and committed:
1. **Always** update the features/roadmap document — add new decisions, update planned/built features if scope changed
2. **If new entities, fields, or enums were designed** — update or create domain model documents
3. **If new or changed REST endpoints** — update endpoint documents
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

## TODO.md

`TODO.md` in the project root tracks deferred items, quick reminders, and topics that surfaced during planning but are out of scope for the current task. **Check it before starting work** — it may contain context or constraints relevant to what you're building.

During planning sessions, if a topic comes up that is out of scope:
- Add it to `TODO.md` with a brief description and which session surfaced it
- Do not let it derail the current design — just capture it and move on
```

- [ ] **Step 2: Commit**

```bash
git add install/.llm/LLM.md
git commit -m "feat: add genericized LLM.md as single entry point for memory bank instructions"
```

---

### Task 4: Write the genericized memory-bank.md

**Files:**
- Create: `install/.llm/instructions/runtime/memory-bank.md`

Same as existing `memory-bank.md` but with Spring-specific references removed (`docs/openapi.json`, `#spring_security` tag examples, `src/main/java` paths, superpowers-specific `-done.md` convention).

- [ ] **Step 1: Write memory-bank.md**

Write `install/.llm/instructions/runtime/memory-bank.md` with this content:

```markdown
The memory bank is a section of the repository (committed alongside other code) that serves as long-term memory across multiple sessions and multiple developers. It is the **first place to look** before re-reading source files — if the answer is in the memory bank, use it.

## Structure

The memory bank lives at `.llm/memory/`. Read `.llm/memory/INDEX.md` for the full directory structure, section descriptions, and document inventory. INDEX.md is the single source of truth for what exists in the memory bank — do not assume a fixed set of folders.

Subdirectories within each category are permitted when a topic grows large enough.

## Document Format

Documents are Markdown with YAML front matter:

` ` `markdown
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
` ` `

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
` ` `
> **Last synced at:** `<short-hash>` (<date>) — if current HEAD is far from this, the memory bank may need refreshing.
` ` `
This helps future sessions detect staleness — if HEAD has moved significantly past the marker, the memory bank may need review.

### Retiring Documents
- If a decision is reversed, update the document to reflect the new decision (don't delete — the history of "we tried X and switched to Y" is valuable)
- If a technique is obsolete (code deleted), archive by adding `[ARCHIVED]` to the title and a note explaining why
```

**Note:** The triple backticks inside the markdown code blocks above are written with spaces between them (` ` `) to avoid escaping issues. When writing the actual file, use proper triple backticks (```).

- [ ] **Step 2: Commit**

```bash
git add install/.llm/instructions/runtime/memory-bank.md
git commit -m "feat: add genericized memory-bank.md instructions"
```

---

### Task 5: Copy unchanged instruction files

**Files:**
- Create: `install/.llm/instructions/runtime/scratchpad.md`
- Create: `install/.llm/instructions/runtime/memory-bank-searching.md`
- Create: `install/.llm/instructions/mcp/context7.md`

These files are already generic and need no changes.

- [ ] **Step 1: Copy scratchpad.md**

Copy the content of `.llm/instructions/runtime/scratchpad.md` to `install/.llm/instructions/runtime/scratchpad.md` verbatim.

- [ ] **Step 2: Copy memory-bank-searching.md**

Copy the content of `.llm/instructions/runtime/memory-bank-searching.md` to `install/.llm/instructions/runtime/memory-bank-searching.md` verbatim.

- [ ] **Step 3: Copy context7.md**

Copy the content of `.llm/instructions/mcp/context7.md` to `install/.llm/instructions/mcp/context7.md` verbatim.

- [ ] **Step 4: Commit**

```bash
git add install/.llm/instructions/
git commit -m "feat: add scratchpad, memory-bank-searching, and context7 instructions"
```

---

### Task 6: Copy skills to install/.claude/skills/

**Files:**
- Create: `install/.claude/skills/sync-memory-bank/SKILL.md`
- Create: `install/.claude/skills/find-best-practices/SKILL.md`

Both skills are already generic and need no changes.

- [ ] **Step 1: Copy sync-memory-bank SKILL.md**

Copy the content of `.claude/skills/sync-memory-bank/SKILL.md` to `install/.claude/skills/sync-memory-bank/SKILL.md` verbatim.

- [ ] **Step 2: Copy find-best-practices SKILL.md**

Copy the content of `.claude/skills/find-best-practices/SKILL.md` to `install/.claude/skills/find-best-practices/SKILL.md` verbatim.

- [ ] **Step 3: Commit**

```bash
git add install/.claude/skills/
git commit -m "feat: add sync-memory-bank and find-best-practices skills"
```

---

### Task 7: Copy agents to install/.claude/agents/

**Files:**
- Create: `install/.claude/agents/memory-searcher-v1.md`
- Create: `install/.claude/agents/researcher-v1.md`

Both agents are already generic and need no changes.

- [ ] **Step 1: Copy memory-searcher-v1.md**

Copy the content of `.claude/agents/memory-searcher-v1.md` to `install/.claude/agents/memory-searcher-v1.md` verbatim.

- [ ] **Step 2: Copy researcher-v1.md**

Copy the content of `.claude/agents/researcher-v1.md` to `install/.claude/agents/researcher-v1.md` verbatim.

- [ ] **Step 3: Commit**

```bash
git add install/.claude/agents/
git commit -m "feat: add memory-searcher-v1 and researcher-v1 agents"
```

---

### Task 8: Write minimal settings.local.json

**Files:**
- Create: `install/.claude/settings.local.json`

Only permissions needed for the memory bank skills and agents. No Spring Boot, Docker, Maven, or framework-specific permissions.

- [ ] **Step 1: Write settings.local.json**

Write `install/.claude/settings.local.json` with this content:

```json
{
  "permissions": {
    "allow": [
      "Bash(git:*)",
      "Bash(ls:*)",
      "Bash(mkdir:*)",
      "Bash(find:*)",
      "Read(**)",
      "Write(**)",
      "WebSearch",
      "WebFetch(domain:github.com)",
      "WebFetch(domain:raw.githubusercontent.com)",
      "WebFetch(domain:api.github.com)",
      "Skill(sync-memory-bank)"
    ]
  }
}
```

- [ ] **Step 2: Commit**

```bash
git add install/.claude/settings.local.json
git commit -m "feat: add minimal settings.local.json with memory bank permissions"
```

---

### Task 9: Write README.md

**Files:**
- Create: `README.md`

The README serves as human documentation AND as Claude's installation script. The "Setup Instructions (for Claude)" section must be clear enough for Claude to follow step by step.

- [ ] **Step 1: Write README.md**

Write `README.md` with this content:

```markdown
# Claude Memory Bank

A persistent memory bank system for [Claude Code](https://claude.ai/code). Gives Claude long-term memory across sessions by storing architecture decisions, domain models, endpoints, business rules, and implementation patterns in a structured, searchable knowledge base committed alongside your code.

## What You Get

- **Memory bank** (`.llm/memory/`) — structured knowledge base with sections for domains, endpoints, decisions, overviews, techniques, product, and infrastructure
- **`/sync-memory-bank` skill** — automatically builds or updates the memory bank from your codebase (full sync for new projects, incremental sync for ongoing work)
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

` ` `bash
curl -L https://github.com/andriiblyzniuk/claude-memory-bank/archive/main.tar.gz \
  | tar -xz --strip-components=1 -C . claude-memory-bank-main/install/
` ` `

This places `.llm/` and `.claude/` directly in the project root.

### Step 2: Handle settings.local.json

Check if `.claude/settings.local.json` already exists in the project:

- **If it does NOT exist:** The downloaded file is already in place. No action needed.
- **If it DOES exist:** Merge the permissions from the downloaded file into the existing one. Read both files, combine the `permissions.allow` arrays (deduplicate), and write the merged result. Do NOT overwrite the existing file.

### Step 3: Handle CLAUDE.md

Check if `CLAUDE.md` exists in the project root:

- **If it does NOT exist:** Create `CLAUDE.md` with this content:
  ` ` `markdown
  @.llm/LLM.md
  ` ` `
- **If it DOES exist:** Append `@.llm/LLM.md` to the end of the file (on a new line), unless it already contains that import.

### Step 4: Initialize the memory bank

Run `/sync-memory-bank` to populate the memory bank from the existing codebase. For greenfield projects with no code yet, skip this step — the memory bank will be populated as the project grows.

### Step 5: Commit

Stage and commit all changes:

` ` `bash
git add .llm/ .claude/ CLAUDE.md
git commit -m "feat: add claude memory bank system"
` ` `

## Usage

### Memory Bank

Claude automatically checks `.llm/memory/INDEX.md` before exploring the codebase. The memory bank gets updated after planning sessions and implementation work.

To manually sync the memory bank after making changes outside of Claude Code:

> /sync-memory-bank

### Finding Best Practices

To discover and install community-vetted Claude Code skills for your tech stack:

> /find-best-practices

Or specify a technology directly:

> /find-best-practices react

### Directory Structure

After installation, your project will have:

` ` `
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
│   │   └── techniques/               # Complex cross-module flows
│   └── scratchpad/                    # Temporary storage (gitignored)
└── .claude/
    ├── settings.local.json            # Permissions for skills
    ├── skills/
    │   ├── sync-memory-bank/SKILL.md  # Memory bank sync skill
    │   └── find-best-practices/SKILL.md  # Best practices finder
    └── agents/
        ├── memory-searcher-v1.md      # Memory bank search agent
        └── researcher-v1.md           # Comprehensive research agent
` ` `

## License

MIT
```

**Note:** The triple backticks inside the markdown code blocks above are written with spaces (` ` `) to avoid escaping issues. When writing the actual file, use proper triple backticks.

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "feat: add README with installation instructions for Claude and humans"
```

---

### Task 10: Add LICENSE and root .gitignore

**Files:**
- Create: `LICENSE`
- Create: `.gitignore` (or update if exists)

- [ ] **Step 1: Write LICENSE**

Write `LICENSE` with the MIT license text, copyright `2026 Andrii Blyzniuk`.

- [ ] **Step 2: Update .gitignore**

Write `.gitignore` with:

```
.claude/
.idea/
.DS_Store
*.swp
*.swo
```

This ensures the repo's own `.claude/` directory (with local development settings) is not committed, while the `install/.claude/` directory IS committed (it's the distributable content).

- [ ] **Step 3: Commit**

```bash
git add LICENSE .gitignore
git commit -m "chore: add MIT license and root .gitignore"
```

---

### Task 11: Clean up old files from project root

**Files:**
- Delete: `.claude/skills/swagger-endpoint-documentation/SKILL.md`
- Delete: `.claude/skills/spring-reactive-logging/SKILL.md`
- Delete: `.claude/skills/http-test-files-with-endpoints/SKILL.md`
- Delete: `.claude/skills/swagger-endpoint-documentation/` (directory)
- Delete: `.claude/skills/spring-reactive-logging/` (directory)
- Delete: `.claude/skills/http-test-files-with-endpoints/` (directory)

The root `.llm/`, `.claude/`, `CLAUDE.md`, and `TODO.md` files remain as-is — they are this repo's own working configuration, not part of the distributable package.

- [ ] **Step 1: Remove Spring Boot skills**

```bash
rm -rf .claude/skills/swagger-endpoint-documentation
rm -rf .claude/skills/spring-reactive-logging
rm -rf .claude/skills/http-test-files-with-endpoints
```

- [ ] **Step 2: Commit**

```bash
git add -A .claude/skills/
git commit -m "chore: remove Spring Boot-specific skills from repository"
```

---

### Task 12: Verify final structure

- [ ] **Step 1: Verify install/ directory has the correct structure**

```bash
find install/ -type f | sort
```

Expected output:
```
install/.claude/agents/memory-searcher-v1.md
install/.claude/agents/researcher-v1.md
install/.claude/settings.local.json
install/.claude/skills/find-best-practices/SKILL.md
install/.claude/skills/sync-memory-bank/SKILL.md
install/.llm/.gitignore
install/.llm/LLM.md
install/.llm/instructions/mcp/context7.md
install/.llm/instructions/runtime/memory-bank-searching.md
install/.llm/instructions/runtime/memory-bank.md
install/.llm/instructions/runtime/scratchpad.md
install/.llm/memory/.keep
install/.llm/memory/INDEX.md
install/.llm/memory/decisions/.keep
install/.llm/memory/domains/.keep
install/.llm/memory/endpoints/.keep
install/.llm/memory/infrastructure/.keep
install/.llm/memory/overviews/.keep
install/.llm/memory/product/.keep
install/.llm/memory/techniques/.keep
```

- [ ] **Step 2: Verify no Spring Boot references in install/ directory**

```bash
grep -r "spring\|Spring\|mvnw\|openapi.json\|src/main/java" install/ || echo "No Spring references found - PASS"
```

Expected: "No Spring references found - PASS"

- [ ] **Step 3: Verify Spring Boot skills are removed from root**

```bash
ls .claude/skills/
```

Expected: only `sync-memory-bank` and `find-best-practices` directories.
