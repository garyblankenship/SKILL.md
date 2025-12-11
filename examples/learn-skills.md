# Example: The /learn Command

**What if Claude could teach itself?**

That's what `/learn` does. Point it at any URL or file—framework docs, API references, code examples—and watch it extract the good stuff, match it to your existing skills, and propose targeted enhancements. No more manually updating your knowledge base. No more stale skills that don't reflect the latest patterns.

This example shows the full chain: a 26-line command that spawns an agent that loads a skill that orchestrates the entire workflow. Three layers, each doing one thing well. The command delegates. The agent orchestrates. The skill contains the methodology.

**Why this matters:** Most Claude Code setups have static prompts that rot over time. `/learn` makes your configuration self-improving. Read the SvelteKit 5 migration guide? Your `sveltekit-patterns` skill now knows about runes. Found a great article on Go error handling? Your `go-specialist` agent just got smarter.

---

## /learn Command

**Source:** `~/.claude/commands/learn.md`

The entry point. Just 26 lines. Does nothing but delegate to the specialist agent. This is the thin-command pattern—commands are triggers, not containers for logic.

~~~markdown
---
description: Interactive skill enhancement from external sources (URLs, files)
argument-hint: <file-path-or-url>
allowed-tools: Task(skill-learning-specialist)
---

# /learn - Skill Enhancement

Delegate to skill-learning-specialist for all orchestration.

**Invoke agent:**
Task(skill-learning-specialist):
  Source: $ARGUMENTS (or ask user if empty)

  Full orchestration:
  - Load skill-learning methodology
  - Fetch/read source content
  - Extract insights (Tier 2-4 only)
  - Match to existing skills
  - Show diff previews
  - Apply enhancements automatically
  - Offer new skill creation for unmatched
  - Display summary and exit
~~~

---

## skill-learning-specialist Agent

**Source:** `~/.claude/agents/skill-learning-specialist.md`

The orchestrator. When the command fires, this agent takes over. It doesn't contain the methodology—that lives in the skill. Instead, it handles the messy real-world stuff: fetching URLs (with Jina fallback when sites block bots), caching skill files for fast matching, managing the approval loop.

Notice the YAML frontmatter. Triggers like "learn from" and "enhance skill" mean this agent can activate proactively—you don't always need the slash command.

~~~markdown
---
name: skill-learning-specialist
description: Orchestrate skill enhancement from external knowledge sources. Scrapes URLs/files, applies novelty-detection to filter training data, matches insights to existing skills, proposes diff previews for user approval. Use PROACTIVELY when user provides documentation URLs, code examples, or mentions learning/improving skills.
tools: Read, WebFetch, Grep, Glob, Skill, Write, Edit, MultiEdit, Bash, AskUserQuestion
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

## When to Use Agent vs Skill

**Use this agent** for:
- Interactive learning sessions (URL/file → skill enhancement)
- Batch processing multiple sources
- Managing user approval flow

**Use `skill-learning` skill directly** for:
- Understanding the methodology
- Manual insight extraction
- Quick reference on matching/enhancement

## Workflow

### 1. Initialize Session
skills_cache = Glob("skills/*/SKILL.md")
# Cache skill names + descriptions for matching

### 2. Process Source
if (input.startsWith("http")) {
  content = WebFetch(input, "Extract technical patterns...")
  if (blocked) {
    content = WebFetch("https://r.jina.ai/" + input, ...)
  }
} else {
  content = Read(input)
}

### 3. Extract & Match (delegate to skill methodology)
- Apply novelty-detection
- Score against cached skills
- Generate enhancement proposals

### 4. Apply Enhancements
For each enhancement:
Present diff preview
Edit skill file immediately
Report: "Applied to {skill}"

### 5. Complete
Display session summary
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

- [ ] Zero Tier 1 insights applied
- [ ] Diff shown for each edit
- [ ] Source attribution in comments
- [ ] Skills stay under 5000 words
~~~

---

## skill-learning Skill

**Source:** `~/.claude/skills/skill-learning/SKILL.md`

The brain. This is where the actual methodology lives—a 7-phase workflow that turns raw documentation into targeted skill enhancements.

The secret sauce is the novelty-detection integration. Not everything in a doc is worth learning. Tier 1 content (stuff Claude already knows from training) gets filtered out. Only Tier 2-4 insights make it through: implementation specifics, architectural trade-offs, counter-intuitive gotchas. The stuff that actually makes your skills better.

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

### File Sources
Single file: Read(file_path)
Directory: Glob("*.md", path) + parallel Read
Code files: Extract comments, docstrings, error handling patterns

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
| 1. Source | WebFetch/Read | Content extracted? |
| 2. Extract | novelty-detection | >=3 Tier 2-4 insights? |
| 3. Match | Glob + score | Any score >=40? |
| 4. Propose | Draft + CLEAR | Validation passes? |
| 5. Preview | Show diff | User understands? |
| 6. Apply | Edit | User approves? |
| 7. Loop | Next source | Continue or done? |
~~~
