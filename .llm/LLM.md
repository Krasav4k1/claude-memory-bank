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
