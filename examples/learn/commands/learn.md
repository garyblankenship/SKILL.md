---
description: Interactive skill enhancement from external sources (URLs, files, directories)
argument-hint: <file-path-url-or-directory>
allowed-tools: Task(skill-learning-specialist)
---

# /learn - Skill Enhancement

This is a thin trigger command. Do not attempt to process the source yourself. You must immediately delegate everything to the specialized agent.

**ACTION:** Use the `Task` tool to invoke the `skill-learning-specialist` agent. 

**Prompt for the Agent:**
"Please process the following source for skill extraction and enhancement: `$ARGUMENTS`. If no arguments were provided, please ask the user what source they would like to learn from."

## Examples
The user can pass arguments directly:
- `/learn https://svelte.dev/docs/kit`
- `/learn ~/docs/architecture.md`
- `/learn ~/.claude/plugins/marketplaces/`
- `/learn ~/projects/my-app`
