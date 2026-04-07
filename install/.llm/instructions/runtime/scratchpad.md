# Scratchpad Usage Guidelines

The directory `.llm/scratchpad` provides temporary storage for preserving context across conversations.

## Creating a Scratchpad

- For each new conversation or task, create a dedicated directory:
  ```bash
  mkdir -p ".llm/scratchpad/$(date +%s)-${TASK_NAME}"
  ```
  Example: `.llm/scratchpad/1692847502-api_design_review`

## Appropriate Usage

- **Use for:** Complex context, detailed analysis, interim results, reference material
- **Don't use for:** Final code, production documents, or anything that should be committed

## Best Practices

1. **Naming Convention:** Use timestamps and descriptive names for easy identification
2. **Content Structure:** Include a README.md in each scratchpad with:
   - Task description
   - Key decisions/considerations
   - Links to relevant files/conversations
3. **Cross-referencing:** Reference scratchpad content with clear file paths

## Context Recovery

If a conversation appears to continue from previous work:
1. List available scratchpads with `ls -la .llm/scratchpad/`
2. Identify likely relevant scratchpads (max 3)
3. Confirm with user which scratchpad to reference

## Maintenance

- Delete scratchpads older than 7 days unless explicitly preserved
- Mark high-value scratchpads with `.keep` file to prevent auto-cleanup

Remember: Scratchpad is for temporary storage only. Transfer any valuable content to permanent files when complete.
