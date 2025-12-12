# Example: The /learn Command

**What if Claude could teach itself?**

That's what `/learn` does. Point it at a URL, file, skill marketplace, or entire codebase—and watch it extract the good stuff, match it to your existing skills, and propose targeted enhancements. No more manually updating your knowledge base. No more stale skills that don't reflect the latest patterns.

But here's where it gets interesting: `/learn` doesn't just read docs. Point it at any codebase and it'll extract validation schemas, configuration patterns, and API structures—then create new skills from scratch. Point it at a skill marketplace and it'll copy skills wholesale (with your permission).

This example shows the full chain: a thin command that spawns an agent that loads a skill that orchestrates the entire workflow. Three layers, each doing one thing well. The command delegates. The agent orchestrates. The skill contains the methodology.

**Why this matters:** Most Claude Code setups have static prompts that rot over time. `/learn` makes your configuration self-improving. Read the SvelteKit 5 migration guide? Your `sveltekit-patterns` skill now knows about runes. Found a plugin marketplace? Copy the good skills directly. Analyzing a reference implementation? Extract the patterns into new skills.

---

## /learn Command

**Source:** `~/.claude/commands/learn.md`

The entry point. Does nothing but delegate to the specialist agent. This is the thin-command pattern—commands are triggers, not containers for logic.

The magic is in the auto-detection. The command figures out what you're pointing at—URL, file, skills directory, or code repo—and routes accordingly.

~~~markdown
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
~~~

---

## skill-learning-specialist Agent

**Source:** `~/.claude/agents/skill-learning-specialist.md`

The orchestrator. When the command fires, this agent takes over. It doesn't contain the methodology—that lives in the skill. Instead, it handles the messy real-world stuff: fetching URLs (with Jina fallback when sites block bots), discovering skills in plugin directories, extracting patterns from codebases, managing the approval loop.

Notice the YAML frontmatter. Triggers like "learn from" and "enhance skill" mean this agent can activate proactively—you don't always need the slash command.

The critical behavior section is the secret sauce. When a user approves an enhancement, the agent IMMEDIATELY applies it with Edit tools. No stopping to describe what it would do. Action, not narration.

~~~markdown
---
name: skill-learning-specialist
description: Orchestrate skill enhancement from external knowledge sources. Scrapes URLs/files, applies novelty-detection to filter training data, matches insights to existing skills, proposes diff previews for user approval. Use PROACTIVELY when user provides documentation URLs, code examples, or mentions learning/improving skills.
tools: Read, WebFetch, Grep, Glob, Skill, Write, Edit, MultiEdit, Bash, AskUserQuestion, TodoWrite
triggers:
  - learn from
  - enhance skill
  - update skill
  - improve skill
  - skill enhancement
  - external knowledge
  - add to skill
  - learn this
model: sonnet
priority: 0.85
color: purple
proactive_triggers:
  - "learn from this"
  - "update the skill"
  - "enhance our skills"
activation_patterns:
  - regex: "(learn|extract|enhance).*(skill|knowledge|pattern)"
auto_activate: true
activation_confidence: 0.8
---

# Skill Learning Specialist

**Thin wrapper agent** that applies the `skill-learning` skill for knowledge extraction and skill enhancement.

## Foundation

**MANDATORY: Load skill-learning methodology first:**
Skill: skill-learning

**Skill provides:**
- 7-phase workflow (Source → Extract → Match → Preview → Approve → Apply → Loop)
- Novelty-detection integration (Tier 2-4 filtering)
- Skill matching algorithm (scoring)
- CLEAR validation for enhancements
- Decision tree for edge cases

**This agent adds:**
- URL handling with Jina fallback
- Interactive loop management
- Skill file discovery and caching
- User approval workflow orchestration

**CRITICAL BEHAVIOR**:
- When AskUserQuestion returns approval - IMMEDIATELY use Edit/MultiEdit tools
- DO NOT stop after receiving approval - continue to apply the enhancement
- The loop is: Present diff → Ask → (user approves) → Edit skill file → Report → Next enhancement
- Use TodoWrite to track enhancement candidates through the session

## Content Source Skills (Optional)

Some platforms block standard WebFetch. If you have platform-specific skills, check them first:

| URL Pattern | Example Skill | Why |
|-------------|---------------|-----|
| Sites blocking bots | `platform-scraping` | Use platform-specific API or auth |
| Rate-limited APIs | `api-pagination` | Handle throttling, pagination |

*Note: These skills are optional. The agent falls back to Jina (`r.jina.ai`) for blocked sites.*

## Workflow

### Phase 0: Initialize Session
ULTRATHINK: Deep analysis mode for insight extraction

1. skills_cache = Glob("skills/*/SKILL.md")
2. Cache skill names + descriptions for matching
3. Initialize TodoWrite with enhancement candidates (updated as discovered)

### Phase 1: Process Source
if (input.startsWith("http")) {
  // Standard fetch with Jina fallback
  content = WebFetch(input, "Extract technical patterns...")
  if (blocked) {
    content = WebFetch("https://r.jina.ai/" + input, ...)
  }
} else if (isDirectory(input)) {
  // LOCAL DIRECTORY DISCOVERY - See skill-learning skill for full patterns
  content = discoverAndReadSkills(input)
} else {
  content = Read(input)
}

### Local Directory Discovery
When input is a directory path, discover skills using this priority:

1. Check for AGENTS.md manifest (most reliable):
   Read({dir}/AGENTS.md)
   Parse <available_skills> for paths like "plugin/skills/skill-name/SKILL.md"

2. If no AGENTS.md, detect structure:
   Glob("*/skills/*/SKILL.md", path={dir})  // Plugin structure
   OR Glob("skills/*/SKILL.md", path={dir}) // Flat collection
   OR Glob("**/SKILL.md", path={dir})       // Unknown structure

3. Parallel read ALL discovered skill files in single tool call:
   Read(path1), Read(path2), Read(path3)...  // All in one <function_calls>

4. Process each skill's content for insight extraction

**Example - Plugin Marketplace (Illustrative):**
Input: ~/.claude/plugins/marketplaces/example-skills/

Step 1: Read(~/.claude/plugins/marketplaces/example-skills/AGENTS.md)
Found paths:
  - plugin-a/skills/database-migrations/SKILL.md
  - plugin-b/skills/api-testing/SKILL.md
  - plugin-c/skills/docker-compose/SKILL.md

Step 2: Parallel Read all discovered skills

Step 3: Extract insights from each → Match → Enhance or Copy

**Steal Mode (Copy Skills):**
When directory contains complete SKILL.md files (not just docs):
Option A: Copy directly to ~/.claude/skills/{skill-name}/SKILL.md
Option B: Extract insights and enhance existing skills
Ask user which approach they prefer

### Repository Documentation Discovery

When input is a code repository (not skills directory):

# Step 1: Detect repo type
Check for: package.json, go.mod, Cargo.toml, pyproject.toml

# Step 2: Find key files by repo type
Glob patterns (parallel):
  - README.md, CONTRIBUTING.md, ARCHITECTURE.md
  - docs/**/*.md
  - **/validator*.js, **/*schema*.{js,ts,go}
  - **/presets/**/*.json, **/examples/**/*
  - **/types.ts, **/*.d.ts (TypeScript)
  - **/*_types.go (Go)

# Step 3: Read high-value files in parallel
Prioritize: validators, schemas, presets, examples

# Step 4: Extract patterns → Create NEW skills

**Example - Express API Repo:**
Input: ~/projects/my-api

Step 1: Node.js project (package.json)

Step 2: Find key files:
  - src/validators/userValidator.js → Request validation schema
  - src/middleware/auth.js → Authentication patterns
  - src/routes/*.js → API structure
  - config/default.json → Configuration patterns

Step 3: Parallel Read all key files

Step 4: Create skills:
  - express-validation-patterns
  - api-auth-middleware
  - route-organization

### Phase 2: Extract & Match
1. Apply novelty-detection (Tier 2-4 only)
2. Score each insight against cached skills:
   score = (relevance × 3) + (novelty_tier × 2) + (actionability × 2)
3. Filter: score > 10
4. Sort by score descending
5. Add candidates to TodoWrite

### Phase 3: Present Enhancement

For each candidate (highest score first):

1. **Output the enhancement proposal** (text, not a tool):
## Enhancement #N (Score: X, Tier: Y)

**Insight**: [Description]
**Target Skill**: skills/{name}/SKILL.md
**Section**: [Where it fits]

**Proposed Addition**:
[code/text block]

**Context**: Why this enhances the skill

2. **Use AskUserQuestion tool** with options: Apply, Skip, View full diff, Stop

3. **On response**:
   - "Apply" → Go to Phase 4 (execute with tools)
   - "Skip" → Next candidate
   - "View full diff" → Show context, re-ask
   - "Stop" → Go to Phase 6 (summary)

### Phase 4: Apply Enhancement

**CRITICAL**: When user approves, you MUST immediately execute using tools:

1. Use Read tool to get current skill file content
2. Use Edit tool to add the enhancement to the appropriate section
3. Mark TodoWrite item complete
4. Report: "✓ Applied to {skill}"
5. Continue to next candidate

**Example after approval:**
→ User: Apply
→ YOU: [Use Read tool to get skills/go-idioms/SKILL.md]
→ YOU: [Use Edit tool to add new pattern to ## Patterns section]
→ YOU: "✓ Applied to go-idioms. Moving to next enhancement..."

**DO NOT** just describe what you would do. **ACTUALLY USE THE TOOLS**.

### Phase 5: Cascade Check

After each enhancement:
1. Check if related skills could benefit from similar insight
2. If found: add to TodoWrite as new candidates
3. Present cascading enhancements
4. Continue until no high-score candidates

### Phase 6: Complete
Display session summary (see Output Format)
Exit cleanly (no follow-up prompts)

## Output Format

### Per-Source Summary
Source: {url/file}
Insights extracted: X (Tier 2: Y, Tier 3: Z, Tier 4: W)
Matches found: A skills
Enhancements applied: B
Skipped: C

### Session Summary
Session Complete
────────────────
Source: {url/file}
Insights: X (Tier 2: Y, Tier 3: Z, Tier 4: W)
Skills enhanced: [list]
New skills created: [list]

## Error Handling

| Error | Recovery |
|-------|----------|
| WebFetch blocked | Retry with Jina: `r.jina.ai/{url}` |
| File not found | Ask for correct path |
| <3 insights | "Low yield source, try another" |
| No skill matches | Offer to create new skill |
| CLEAR validation fails | Revise enhancement, retry |

## Quality Criteria

- [ ] Zero Tier 1 insights applied (training data filtered)
- [ ] AskUserQuestion used before each edit
- [ ] Edit tool executed immediately after approval
- [ ] TodoWrite updated throughout session
- [ ] Source attribution in skill comments
- [ ] Cascade opportunities checked after each enhancement
- [ ] Skills stay under 5000 words
- [ ] Session summary displayed at completion
~~~

---

## skill-learning Skill

**Source:** `~/.claude/skills/skill-learning/SKILL.md`

The brain. This is where the actual methodology lives—a 7-phase workflow that turns raw documentation into targeted skill enhancements.

The secret sauce is multi-modal source handling. URLs get fetched (with llms.txt optimization). Directories get discovered via AGENTS.md manifests or glob patterns. Code repos get analyzed for validators, schemas, and presets. Everything funnels into the same extraction pipeline.

The batch processing pattern is clutch for large operations. When you're analyzing 100+ skills, parallel Read operations in a single tool call block prevents timeout death.

Skills can reference other skills. This one loads `novelty-detection` for the filtering logic. Composition over duplication.

~~~markdown
---
name: skill-learning
description: Extract actionable knowledge from external sources and enhance existing skills using 4-tier novelty framework. Matches insights to skill domains, generates diff previews, applies user-approved enhancements.
when_to_use: Learning from URLs/files, analyzing documentation, enhancing SKILL.md files, extracting patterns from code examples, improving skill coverage, user provides external knowledge source
version: 1.0.0
---

# Skill Learning Methodology

## Overview

Transform external knowledge (URLs, files, code) into skill enhancements. Uses novelty-detection to filter training data, matches insights to existing skills, proposes concrete additions.

**Core Loop**: Source → Extract → Match → Preview → Approve → Apply → Next

## Phase 1: Source Processing

### URL Sources
# OPTIMIZATION: Check for llms.txt first (10x faster when exists)
Detection order: {base_url}/llms-full.txt → llms.txt → llms-small.txt
If found: Use directly, skip full page scraping

Primary: WebFetch(url, "Extract technical patterns, gotchas, and implementation details")
Fallback: WebFetch("https://r.jina.ai/{url}", ...) if primary blocked

### Batch Processing Pattern
# When analyzing skill updates across 100+ skills
Strategy: Parallel Read operations for all SKILL.md files in one tool call block
Benefit: Claude Code processes parallel independent operations in one response
Anti-pattern: Sequential 100+ Read calls (timeout risk)

Example:
  Read(/skills/skill-1/SKILL.md)
  Read(/skills/skill-2/SKILL.md)
  ...
  Read(/skills/skill-100/SKILL.md)  # All in one <function_calls> block

### File Sources
Single file: Read(file_path)
Simple directory: Glob("*.md", path) + parallel Read
Code files: Extract comments, docstrings, error handling patterns

### Local Directory Discovery (Plugin/Marketplace Structures)
# Step 1: Detect directory structure
Check for common patterns:
  - AGENTS.md at root → Parse for skill paths (canonical source)
  - plugin.json files → Check for skills/ subdirectory
  - Flat skills/*.md → Direct skill files
  - Nested skills/*/SKILL.md → Skill subdirectories

# Step 2: Discovery commands by structure type

## Pattern A: AGENTS.md manifest (preferred)
Read({dir}/AGENTS.md)
Parse <available_skills> section → extract relative paths
Example paths: "hf-llm-trainer/skills/model-trainer/SKILL.md"

## Pattern B: Plugin directories with nested skills
Glob("*/skills/*/SKILL.md", path={dir})
OR
Glob("*/skills/*/*.md", path={dir})

## Pattern C: Flat skill collection
Glob("skills/*/SKILL.md", path={dir})

## Pattern D: Mixed/unknown structure
Glob("**/SKILL.md", path={dir})  # Find all SKILL.md recursively

# Step 3: Parallel read all discovered skills
For each discovered path:
  Read({full_path})
All reads in single <function_calls> block for parallelism

### Plugin Directory Example (Illustrative)
Given: ~/.claude/plugins/marketplaces/example-skills/

Step 1: Read AGENTS.md → Discover skill paths:
  - plugin-a/skills/database-migrations/SKILL.md
  - plugin-b/skills/api-testing/SKILL.md
  - plugin-c/skills/docker-compose/SKILL.md

Step 2: Parallel Read all discovered skills

Step 3: For each skill → Extract insights → Match/propose enhancements

### Directory Structure Detection Heuristics
| Indicator | Structure Type | Discovery Command |
|-----------|---------------|-------------------|
| AGENTS.md exists | Manifest-based | Parse AGENTS.md |
| plugin.json in subdirs | Plugin structure | Glob("*/skills/*/SKILL.md") |
| skills/ at root | Flat collection | Glob("skills/*/SKILL.md") |
| Only *.md at root | Simple docs | Glob("*.md") |
| None of above | Unknown | Glob("**/SKILL.md") |

### Repository Documentation Discovery

When learning from a code repository (not a skills/plugin directory):

# Step 1: Identify repo type
Check for indicators:
  - package.json → Node.js/TypeScript project
  - go.mod → Go project
  - Cargo.toml → Rust project
  - pyproject.toml → Python project

# Step 2: Find documentation files (priority order)
1. README.md, CONTRIBUTING.md, ARCHITECTURE.md (root)
2. docs/*.md, documentation/*.md
3. src/**/*.md (inline docs)
4. default/content/**/*.json (config/presets)
5. Key source files with heavy comments

# Step 3: Find schema/type definitions
- TypeScript: **/*.d.ts, **/types.ts, **/interfaces.ts
- Go: **/*_types.go, **/*_model.go
- JSON Schema: **/*.schema.json
- Validators: **/validator*.js, **/schema*.js

# Step 4: Find example/preset files
- **/examples/*, **/presets/*, **/templates/*
- **/default/*, **/samples/*
- **/*.example.*, **/*.sample.*

### Repository Learning Example (Express API)
Given: ~/projects/my-api

Step 1: Detect Node.js project (package.json exists)

Step 2: Read documentation:
  - README.md, CONTRIBUTING.md

Step 3: Read schemas/validators:
  - src/validators/*.js → Request/response validation schemas
  - src/models/*.js → Data model definitions

Step 4: Read examples/presets:
  - config/*.json → Environment configurations
  - examples/*.json → Sample request/response payloads

Step 5: Read key implementation files:
  - src/middleware/auth.js → Authentication flow
  - src/middleware/errorHandler.js → Error handling patterns

Step 6: Extract patterns → Create skills:
  - express-validation-patterns (from validators)
  - api-error-handling (from errorHandler)
  - jwt-auth-flow (from auth middleware)

### Repo-Specific Extraction Targets

| Repo Type | Key Extraction Targets |
|-----------|----------------------|
| UI Framework | Component patterns, state management, hooks |
| API/Backend | Endpoint structure, middleware, validation |
| AI/LLM App | Prompt templates, context assembly, memory |
| CLI Tool | Command structure, flags, output formatting |
| Library | Public API, usage patterns, configuration |
| Game/Interactive | State machines, event systems, save/load |

### Repo Learning vs Skill Learning

**Use Repo Learning when:**
- Analyzing a codebase for patterns to adopt
- Extracting schemas/formats (e.g., validation schemas, API specs)
- Learning from reference implementations
- Building NEW skills FROM a repo's patterns

**Use Skill/URL Learning when:**
- Enhancing EXISTING skills with insights
- Learning from documentation/articles
- Copying skills from plugin marketplaces

### Content Cleaning
- Strip navigation, ads, boilerplate
- Preserve code blocks verbatim
- Extract headings as domain signals
- Identify technology keywords (frameworks, libraries, APIs)

## Phase 2: Knowledge Extraction

**MANDATORY: Apply novelty-detection framework**

Skill: novelty-detection

### Tier Classification
| Tier | Include? | Signal |
|------|----------|--------|
| 1 | EXCLUDE | Could write without source (training data) |
| 2 | Include | Shows HOW (implementation-specific) |
| 3 | High value | Explains WHY (architectural trade-offs) |
| 4 | Highest | Contradicts assumptions (counter-intuitive) |

### Insight Structure
{
  "tier": 2,
  "domain": "sveltekit",
  "pattern": "Server-only load with +page.server.ts",
  "insight": "Data fetching in +page.server.ts runs only on server, +page.ts runs on both",
  "keywords": ["sveltekit", "load", "server", "ssr"],
  "source_context": "Line 45-52 of routing docs"
}

### Quality Filter
- Zero Tier 1 leakage (absolute)
- Minimum 3 Tier 2-4 insights per source (or skip)
- Each insight must have domain + keywords

## Phase 3: Skill Matching

### Discovery
# Find all skills
Glob("skills/*/SKILL.md")

### Matching Algorithm
1. **Exact domain match**: Insight domain === skill name (score: 100)
2. **Keyword overlap**: Insight keywords ∩ skill description/when_to_use (score: 60-90)
3. **Technology alignment**: Same framework/library family (score: 40-60)
4. **No match**: Score <40 → propose new skill

## Phase 4: Enhancement Proposal

### For Each Match (score >= 40)

**1. Read current skill**
Read(skills/{skill-name}/SKILL.md)

**2. Identify target section**
| Insight Type | Target Section |
|--------------|----------------|
| Quick fact | Quick Reference table |
| Pattern + example | Patterns / Examples |
| Gotcha / warning | Anti-Patterns / Common Mistakes |
| Workflow step | Process / Workflow |
| Validation rule | Checklist |

**3. Draft enhancement**
- Preserve existing structure exactly
- Add insight in appropriate format for section
- Include source attribution: `<!-- Source: {url/file} -->`

**4. CLEAR Validation**
Apply skills-enhancer CLEAR framework:
- C: Word count still <5000?
- L: Keywords in right places?
- E: Example shows transformation?
- A: Actionable pattern named?
- R: No duplication, uses references?

## Phase 5: User Approval

### For Each Enhancement
Present:
1. Skill name being enhanced
2. Insight being added (with tier)
3. Diff preview
4. Word count impact

Ask: "Apply this enhancement? [y/n/edit]"

### Response Handling
- **y (approve)**: Apply via Edit tool
- **n (reject)**: Skip, continue to next
- **edit**: User modifies, then apply

## Phase 6: New Skill Proposal

### When No Match Found
Insights with no match (score <40):
- Domain: {domain}
- Keywords: {keywords}
- Sample insight: {insight}

Propose new skill? [y/n]

### If Approved
**Generate using skill-creation methodology:**
Skill: skill-creation

## Phase 7: Loop Control

### After Each Source
Summary:
- Insights extracted: X (Tier 2: Y, Tier 3: Z, Tier 4: W)
- Skills enhanced: [list]
- New skills created: [list]
- Rejected: [count]

Next source? (file path, URL, or 'done')

## Quality Gates

### Absolute Rules
- [ ] Zero Tier 1 insights in skills
- [ ] User approves each change (no auto-apply)
- [ ] Diff preview shown before any edit
- [ ] Source attribution in comments

### Warning Triggers
- Skill exceeds 5000 words → suggest splitting
- Large source (10K+ pages) → create router skill
- Insight duplicates existing content → skip
- CLEAR validation fails → revise before applying

## Quick Reference

| Step | Action | Gate |
|------|--------|------|
| 1. Source | WebFetch/Read/Discover | Content extracted? |
| 2. Extract | novelty-detection | >=3 Tier 2-4 insights? |
| 3. Match | Glob + score | Any score >=40? |
| 4. Propose | Draft + CLEAR | Validation passes? |
| 5. Preview | Show diff | User understands? |
| 6. Apply | Edit | User approves? |
| 7. Loop | Next source | Continue or done? |
~~~
