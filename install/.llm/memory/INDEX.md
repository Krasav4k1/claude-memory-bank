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
