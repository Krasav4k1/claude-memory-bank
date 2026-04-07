---
name: sync-memory-bank
description: Synchronizes the memory bank with current codebase state — detects staleness via git watermark and runs full or incremental sync
model: inherit
color: blue
---

# Sync Memory Bank Agent

Synchronize the `.llm/memory/` knowledge base with the current state of the codebase. Automatically determines whether to run a full build or an incremental update. Return a concise summary of what was updated.

**Your goal is MAXIMUM COVERAGE.** The memory bank must be a complete, precise reference for this project. Every module, every domain, every significant flow must be documented. A future Claude session should be able to understand this entire project from the memory bank alone without reading source files. Do not cut corners. Do not leave modules undocumented. Do not merge unrelated domains into a single file to save effort.

## Mode Selection

Read `.llm/memory/INDEX.md`. Based on what you find, pick one mode:

- **FULL SYNC** — INDEX.md doesn't exist, is empty, or has no documents listed
- **INCREMENTAL SYNC** — INDEX.md has documents + git watermark exists + `git diff --name-only <hash>..HEAD` shows changes
- **UP TO DATE** — watermark exists + no changes since watermark → report "memory bank is up to date" and stop
- **SPOT-CHECK** — INDEX.md has documents but no watermark, or project is not a git repo
- **FULL SYNC (fallback)** — watermark hash not in git history (force-push/rebase)

---

## Full Sync

Build the memory bank from scratch. **Read every significant file.** Do not skim. Do not guess from filenames.

### 1. Check Existing State

- If `.llm/memory/INDEX.md` exists, read it for any existing structure
- If `.llm/instructions/runtime/memory-bank.md` exists, read it for document format rules

### 2. Exhaustive Codebase Analysis

Analyze the project thoroughly by actually reading the source files — not just listing directories. What you look for depends on what the project actually is — do NOT apply a fixed backend checklist to every project.

**Phase 1 — Project identification:**
- List top-level directories, identify module/package layout
- Read config files to identify language, version, framework, build tool, package manager
- Read the dependency manifest (package.json, pom.xml, build.gradle, go.mod, Cargo.toml, etc.) — extract ALL significant dependencies and their versions
- Find and read entry points (main class, app bootstrap, index file, router setup)

**Phase 2 — Module-by-module deep dive:**

For EVERY module/package/directory that contains business logic, do ALL of the following:
- **Read entity/model files** — extract every field, type, relationship, validation, default value, enum values. Do not summarize — document precisely
- **Read service/logic files** — understand what each service does, what it calls, what events it publishes/listens to, what business rules it enforces
- **Read controller/handler/route files** — extract every endpoint/route with HTTP method, path, auth requirement, permissions, request/response shape
- **Read repository/data-access files** — note custom queries, joins, indexes, computed columns
- **Read test files** — understand test strategy, what's covered, test utilities, mock setup
- **Read middleware/filters/interceptors** — auth checks, CORS, rate limiting, error handling, request transformation
- **Read configuration files** — profiles, environment-specific config, feature flags

**Phase 3 — Cross-cutting concerns:**
- Map ALL cross-module communication (events, direct calls, shared interfaces, message queues)
- Document auth flow end-to-end (login → token → validation → refresh → logout)
- Document permission/authorization model completely (roles, permissions, resolution logic)
- Map error handling strategy (error codes, error response shape, exception hierarchy)
- Identify state machines and lifecycle flows (status transitions, guards, side effects)

**Phase 4 — Infrastructure & tooling:**
- Read CI/CD config (GitHub Actions, GitLab CI, etc.)
- Read Docker/container config
- Read deployment scripts/config
- Read database migrations — understand schema evolution
- Read environment/config files for different environments

**Phase 5 — Design documents & specs:**
- Check for `docs/superpowers/` — if it exists, read EVERY file in it. These are design specs and implementation plans. For each spec:
  - Extract the feature scope, design decisions, API contracts, data models
  - Note whether the spec is completed (`-done.md` suffix or `STATUS [DONE]` in title) or pending
  - Cross-reference with the actual implementation — flag any discrepancies
- Check for `docs/PROJECT.md`, `docs/`, `specs/`, or any documentation directory
- Check for OpenAPI/Swagger specs (`openapi.json`, `openapi.yaml`, `swagger.json`)
- Process ALL design documents found — extract decisions, constraints, and planned features into the memory bank

### 3. Determine Memory Sections

**Be maximally proactive.** Based on your exhaustive analysis, create every section this project needs for complete coverage. Do NOT hold back — if there's enough content for a section, create it. Think about what a developer new to this project would need to know, and make sure NOTHING is missing.

Create a `## Section Name` in INDEX.md for each section, with a one-line description. Create the matching directory with `mkdir -p .llm/memory/<section>/`.

**Examples of sections by project type** (mix and match — most projects need a combination):

| Section | When to create | Example content |
|---------|---------------|-----------------|
| `overviews` | Always | Architecture overview, tech stack, module map, naming conventions |
| `decisions` | Always | "We chose X over Y because Z", known gotchas, intentional oddities |
| `product` | Always | Project goal, feature inventory, roadmap, business rules |
| `domains` | Projects with entities/models | Entity fields, enums, relationships, state machines, validation rules |
| `schemas` | Projects with databases | Table definitions, FK/cascade, indexes, constraints, migration patterns |
| `endpoints` | Backend/API projects | REST/GraphQL routes, auth, DTOs, error codes catalog |
| `routes` | Frontend/fullstack with pages | Page map, route guards, layouts, navigation, dynamic routes |
| `components` | Frontend with component architecture | Component catalog, design system, tokens, shared hooks, form patterns |
| `state` | Frontend with state management | Stores, slices, data flow, caching, invalidation strategy |
| `security` | Projects with auth/security | Auth flows end-to-end, RBAC, CORS, token storage, permission model |
| `environment` | Always (if env vars exist) | Env var catalog, profiles, local dev setup, external credentials needed |
| `integrations` | Projects with external services | Third-party APIs, webhooks, scheduled jobs, message queues |
| `infrastructure` | Projects with deployment config | CI/CD, Docker, hosting, reverse proxy, monitoring, DB hosting |
| `techniques` | When complex multi-file flows exist | End-to-end flows, cross-module sequences, complex algorithms |
| `testing` | Projects with test suites | Test strategy, config, mock setup, test utilities, coverage approach |
| `dependencies` | When non-obvious dep behavior exists | Version pins, migration gotchas, known issues |

**Rules:**
- Every project gets `overviews` and `decisions` at minimum
- Name sections to match the project's vocabulary (e.g., `modules` not `domains` if that's what the codebase calls them)
- When in doubt, CREATE the section — more coverage is always better than less

### 4. Write Documents

**ONE DOCUMENT PER MODULE/DOMAIN MINIMUM.** Do not merge unrelated modules into a single document. If the project has 9 modules, the domains section should have at minimum 9 documents (or a justified subset if some modules are trivial).

Each INDEX.md section needs a `## Section Name` header, a description line, and document entries with one-line summaries.

For each document:
1. Create the directory if needed (`mkdir -p .llm/memory/<section>/`)
2. Write with YAML frontmatter (`title`, `tags`)
3. Be thorough — 50-300 lines per document. Include ALL fields, ALL enum values, ALL endpoints, ALL business rules. Do not summarize when you can be precise. Use the space you need — a complex module with 20 fields, 5 enums, and 3 state machines needs more lines than a simple utility
4. Include `## Relevant Code` with specific file paths (not just directories)
5. Include `## Further Reading` with cross-references to related memory bank documents
6. Add to INDEX.md with a one-line summary

**Coverage checklist — verify before finishing. Skip items that don't apply to this project, but do not skip items that apply even partially.**

Project Identity & Product:
- [ ] Project goal / product idea documented — what is this, who is it for, what problem does it solve
- [ ] Product/roadmap section reflects feature status (built, planned, in progress)
- [ ] Business rules, domain invariants, and constraints documented
- [ ] Design specs from `docs/superpowers/` (or similar) processed and cross-referenced with implementation

Architecture & Overview:
- [ ] Architecture overview document exists (module map, communication patterns, entry points)
- [ ] Tech stack document lists ALL significant dependencies with versions
- [ ] Cross-module/cross-package interactions documented (events, direct calls, shared interfaces, message queues, pub/sub)
- [ ] Naming conventions and file organization patterns documented (where new files go, how things are named)
- [ ] Known gotchas or intentional oddities documented (things that look wrong but are deliberate)

Domain & Data (backend, fullstack, or any project with a data layer):
- [ ] Every module/package with business logic has a dedicated domain document
- [ ] Every entity/model with ALL fields, types, defaults, nullable/required, and relationships documented
- [ ] Every enum with ALL its values documented
- [ ] Every state machine / status lifecycle diagrammed with transitions, guards, and side effects
- [ ] Validation rules documented — what's validated, where (controller? service? entity? DB constraint?), and what error is returned

Database Schema & Relations:
- [ ] Table names, primary key strategy (UUID? auto-increment? composite?)
- [ ] Foreign keys, cascade behavior (ON DELETE CASCADE? SET NULL? RESTRICT?)
- [ ] Unique constraints, check constraints, not-null constraints
- [ ] Indexes (unique, partial, composite, full-text, spatial) and why they exist
- [ ] Database migrations history and patterns (tool, naming convention, how to add new migration)
- [ ] Views, materialized views, triggers, stored procedures (if any)

API & Endpoints (backend or API projects):
- [ ] Every controller/handler covered in an endpoint document (one doc per controller or tightly-related group)
- [ ] Every endpoint has: HTTP method, path, auth requirement, permissions, request/response shape
- [ ] Error codes catalog — every error code with when it's returned and HTTP status
- [ ] API conventions documented (error response format, pagination pattern, versioning, headers, rate limits)

Routes & Pages (frontend or fullstack):
- [ ] Complete route/page map with paths, purpose, and auth requirements
- [ ] Route guards, middleware, redirects, protected vs public pages documented
- [ ] Layouts, nested routes, route groups, dynamic routes documented
- [ ] Navigation structure and user flows documented

Components & UI (frontend):
- [ ] Component catalog — every significant component with purpose and key props
- [ ] Design system / tokens / theming documented (colors, typography, spacing, breakpoints)
- [ ] Component patterns documented (server vs client, smart vs dumb, composition)
- [ ] Form handling and validation patterns documented (library, strategy, error display)
- [ ] Shared hooks and utilities catalog — what reusable hooks exist, what they do, when to use them
- [ ] i18n / localization setup documented (if applicable — locales, translation patterns, RTL support)
- [ ] Accessibility patterns documented (ARIA, keyboard nav, screen reader considerations)

State Management (frontend):
- [ ] Every store/slice/context documented with shape, actions, and selectors
- [ ] Data fetching strategy documented (caching, invalidation, optimistic updates, retry)
- [ ] Client state vs server state separation documented

Auth & Security:
- [ ] Auth flow documented end-to-end (login → token → validation → refresh → logout)
- [ ] Permission/authorization model complete (roles, permissions, resolution logic)
- [ ] Security mechanisms documented (CORS, CSP, rate limiting, encryption, token storage)
- [ ] Platform-specific auth differences documented (if multi-platform)

Environment & Configuration:
- [ ] ALL environment variables documented with: name, description, required vs optional, default value, example
- [ ] Environment profiles/modes documented (dev, staging, prod — what differs between them)
- [ ] External service credentials listed (what's needed: API keys, OAuth secrets, DB connection strings)
- [ ] Feature flags documented (if any)
- [ ] Local development setup documented (how to run the project, prerequisites, seed data)

External Integrations:
- [ ] Every third-party service/API integration documented (what service, what for, which module uses it)
- [ ] Webhook handlers documented (inbound and outbound)
- [ ] Scheduled jobs / background tasks / cron documented (what runs, when, what it does)
- [ ] Message queues / event buses (if external) documented

Infrastructure & Deployment:
- [ ] CI/CD pipeline documented (stages, triggers, environments, secrets)
- [ ] Container/Docker setup documented (Dockerfile, compose, multi-stage builds)
- [ ] Deployment architecture documented (servers, CDN, reverse proxy, load balancer, DNS)
- [ ] Database hosting and backup strategy documented
- [ ] Monitoring / logging / alerting setup documented (if present)

Testing:
- [ ] Test strategy documented (unit, integration, e2e — what level covers what)
- [ ] Test configuration, utilities, and helpers documented
- [ ] Mock/stub/fixture setup documented (how external services are mocked, test DB setup)

**Writing priorities:** Architecture overview first → domain models (one per module) → endpoints (one per controller) → decisions → techniques → infrastructure → product.

### 5. Set Watermark

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

### 2. Check for Design Specs

If any changed files are in `docs/superpowers/` or similar documentation directories, read them fully and update memory bank accordingly — extract decisions, new features, API contracts, data model changes.

### 3. Map Changes to Memory Bank Documents

Read INDEX.md section descriptions. For each changed source file, determine which memory bank documents it affects by matching the change type to section descriptions in INDEX.md.

### 4. Update Affected Documents

For each affected document:
1. Read the current memory bank document
2. Read the changed source files — actually read them, don't guess from filenames
3. Edit to reflect current state precisely
4. If a change introduces a new module, domain, or flow that has no document — **create one**. Do not leave gaps.
5. If a change removes or renames something — update or archive the document

### 5. Coverage Check

After updating, scan the memory bank for gaps:
- Any module without a dedicated domain document? Create one.
- Any controller without endpoint documentation? Create one.
- Any new cross-module flow without a technique document? Create one.
- Any new design spec not reflected in product/roadmap? Update it.

### 6. Advance Watermark

Update the "Last synced at" line in INDEX.md to current HEAD.

---

## Spot-Check

1. Read INDEX.md for the full document list
2. Spot-check key claims against current codebase (file paths, entity fields, enum values, versions)
3. Fix any stale information
4. **Check for undocumented modules** — if any module lacks coverage, create the missing documents
5. If git repo, add the watermark

---

## Document Format

Follow `.llm/instructions/runtime/memory-bank.md`. Key rules: YAML frontmatter with title/tags, thorough content, `## Relevant Code` with file paths, `## Further Reading` with cross-references.

## Guidelines

- **INDEX.md is the single source of truth** — read it to discover sections, update it when adding/removing documents
- **Adapt to the project** — create sections that fit what the codebase contains
- **Coverage AND quality** — every module documented, every document accurate
- **One document per module/domain minimum** — do not merge unrelated things
- **Don't duplicate CLAUDE.md**
- **Process ALL design docs** — `docs/superpowers/`, `docs/`, specs, plans — everything feeds into the memory bank

## Error Handling

- **Directory doesn't exist:** Create with `mkdir -p`
- **Watermark hash not in history:** Fall back to full sync
- **INDEX.md malformed:** Rebuild from documents on disk

## Response Format

Return a concise summary:
```
Memory bank synced: [FULL/INCREMENTAL/UP TO DATE]
- Sections: [list of sections created/updated]
- Documents created: [list]
- Documents updated: [list]
- Coverage: [X modules documented / Y total modules]
- Design specs processed: [list from docs/superpowers/ if any]
- Watermark: <old-hash> → <new-hash>
```
