---
description: Interactive skill enhancement from external sources (URLs, files, directories)
argument-hint: <file-path-url-or-directory>
allowed-tools: Task(skill-learning-specialist)
---

# /learn - Skill Enhancement

Delegate to skill-learning-specialist for all orchestration.

**Invoke agent:**
Task(skill-learning-specialist):
  Source: $ARGUMENTS (or ask user if empty)

  Full orchestration:
  - Load skill-learning methodology
  - Detect source type (URL, file, or directory)
  - For directories: discover skills via AGENTS.md or Glob patterns
  - Fetch/read source content (parallel reads for directories)
  - Extract insights (Tier 2-4 only)
  - Match to existing skills
  - Show diff previews
  - Apply enhancements OR copy complete skills (ask user preference)
  - Offer new skill creation for unmatched
  - Display summary and exit

**Source Types:**
- URL: `https://docs.example.com/guide`
- File: `~/docs/patterns.md`
- Skills Directory: `~/.claude/plugins/marketplaces/my-skills/`
- Code Repository: `~/projects/my-app` (extracts from validators, schemas, presets)

**Auto-Detection:**
| Source Pattern | Detection | Action |
|----------------|-----------|--------|
| `http://`, `https://` | URL | WebFetch + extract |
| AGENTS.md or SKILL.md present | Skills dir | Copy or enhance |
| package.json, go.mod, etc | Code repo | Find schemas, presets → create skills |
| Only *.md files | Docs dir | Extract insights → enhance |

**Examples:**
/learn https://svelte.dev/docs/kit      # Learn from URL
/learn ~/docs/architecture.md           # Learn from file
/learn ~/.claude/plugins/marketplaces/  # Copy/enhance from skills
/learn ~/projects/my-app                # Extract patterns from repo
