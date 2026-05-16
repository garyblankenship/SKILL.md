---
name: opencode-learn
description: Extracts actionable knowledge from external sources and enhances existing skills using a 4-tier novelty framework. Use PROACTIVELY when a user says "/learn &lt;source&gt;", provides documentation URLs, code examples, or explicitly asks to extract patterns from a repository or marketplace.
---

# INSTRUCTIONS FOR AI ASSISTANT: The /learn Command

## System Prompt

You are executing a rigid knowledge extraction protocol. Your primary objective is to extract technical patterns that are both **novel** and **behavior-changing**, then inject only the durable ones into the user's local `skills/` or `prompts/` directory.

You are acting as the Command, the Agent, and the Skill all at once. You must orchestrate the fetching, extracting, matching, and applying.

**CRITICAL DIRECTIVE:** You have a natural tendency to summarize everything you read. **DO NOT DO THIS.** The user's context window is precious. You must aggressively filter out "Tier 1" knowledge (things you already know from your pre-training data), and you must also reject low-leverage facts that will not change future agent behavior.

**Core Execution Loop**: 1. Source → 2. Extract → 3. Match → 4. Preview → 5. Approve → 6. Apply → 7. Loop

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| **Summarizing Training Data** | Bloats the context window with useless Tier 1 facts (e.g. "React uses a Virtual DOM"). | Ruthlessly apply the Novelty Test. Exclude Tier 1. |
| **Saving Trivia** | Novel but low-leverage facts make skills longer without changing behavior. | Apply the Leverage Test. Keep only recurring, risky, or workflow-changing details. |
| **Appending Conflicts** | New advice can contradict an older rule, leaving future agents with both. | Detect conflicts and propose replace, qualify, split, or skip. |
| **Invisible Decay** | Skills can become stale without an obvious failure signal. | Track learned source/date metadata and report stale, duplicate, or unattributed sections. |
| **Sequential File Reading** | Calling `read` 100 times in a loop will cause you to time out. | Use **Parallel Tool Calls** inside a single block. |
| **Asking before Editing** | If the user already said "Apply" in Phase 5, pausing to ask permission to edit is maddening. | Execute the edit immediately upon user approval. |
| **Missing Source Links** | Future agents won't know where the pattern came from. | Always append `<!-- Source: {url/file} -->`. |

---

## Phase 1: Source Processing (Execution Steps)

### 1a. URL Sources
**ACTION:** Fetch the URL using a web scraping or fetching tool.
*   **OPTIMIZATION:** Check for `llms.txt` first. Attempt `{base_url}/llms-full.txt` → `llms.txt` → `llms-small.txt`. If found, use it directly to avoid scraping HTML.

### 1b. File Sources & Batch Processing
**ACTION:** If the source is a local directory or repository, use your file search (`glob`/`grep`) and `read` tools.
*   **Strategy:** When analyzing multiple files (e.g., discovering existing skills), you MUST use **Parallel Tool Calls**. Output all your `read` tool calls in a single response.

### 1c. Discovery
**ACTION:** Use your search tools to find existing `SKILL.md` files or `AGENTS.md` manifests.
*   Look for `AGENTS.md` at the project root.
*   Look for `skills/*/SKILL.md` or `prompts/*.md`.
*   Read all discovered files in parallel.

---

## Phase 2: Knowledge Extraction

**MANDATORY:** You must apply a two-axis filter: novelty first, leverage second. A fact is not skill material merely because it is new.

### 2a. Novelty Classification
You must classify every extracted insight into one of four tiers:
| Tier | Include? | Signal |
|------|----------|--------|
| 1 | **EXCLUDE** | Could write without source (training data) |
| 2 | Include | Shows HOW (implementation-specific) |
| 3 | High value | Explains WHY (architectural trade-offs) |
| 4 | Highest | Contradicts assumptions (counter-intuitive) |

### The Novelty Test
For every insight, ask yourself: *"Could I have written this WITHOUT reading the source?"*
*   **If YES** → It is Tier 1. You MUST EXCLUDE IT.
*   **If NO** → Continue to Tier 2-4 classification.

### 2b. Leverage Classification
For every Tier 2-4 insight, ask: *"Will preserving this change future agent behavior in a recurring or high-risk task?"*

| Leverage | Include? | Signal |
|----------|----------|--------|
| 0 | EXCLUDE | Interesting but unlikely to affect future work |
| 1 | Usually exclude | Narrow fact with no clear action or validation impact |
| 2 | Include when compact | Reusable implementation detail, command, flag, schema, or convention |
| 3 | Include | Changes workflow, placement, safety, validation, or debugging behavior |
| 4 | Highest | Prevents a likely failure, contradiction, data loss, security issue, or expensive rework |

### Acceptance Gate
Accept an insight only when:
1. Novelty tier is 2, 3, or 4.
2. Leverage is 2, 3, or 4.
3. The proposed text can be written as an actionable rule, workflow step, validation gate, or source-backed exception.

Reject or defer when:
- The insight is true but does not change what the agent will do.
- The insight belongs in a bulky reference file rather than the active skill body.
- The insight is a one-off local detail without recurring value.
- The insight conflicts with existing guidance and the conflict cannot be resolved clearly.

### Insight Structure Requirements
You must structure each extracted insight logically before scoring it:
```json
{
  "tier": 2,
  "leverage": 3,
  "domain": "sveltekit",
  "pattern": "Server-only load with +page.server.ts",
  "insight": "Data fetching in +page.server.ts runs only on server, +page.ts runs on both",
  "behavior_change": "Prefer +page.server.ts when fetched data must never reach the browser",
  "keywords": ["sveltekit", "load", "server", "ssr"],
  "source_context": "Line 45-52 of routing docs"
}
```

---

## Phase 3: Placement and Skill Matching

### Placement Decision
Before matching to a skill or prompt, decide where the accepted insight belongs:

| Destination | Use When |
|-------------|----------|
| Existing skill or prompt body | The insight should affect frequent agent behavior directly |
| Reference file | The detail is useful but bulky, example-heavy, or rarely needed |
| Project guidance | The rule is local to one repository rather than a reusable domain skill |
| New skill or prompt | The insight opens a recurring domain with no clear existing owner |
| Nowhere | The insight is novel but low leverage, duplicated, stale, or unresolved |

If placement is ambiguous, prefer the smallest durable home. Do not put project-local conventions into global skills unless they generalize.

### Matching Algorithm
You must score each extracted insight against the user's existing skills/prompts to find the best home for it.
1. **Exact domain match**: Insight domain === skill name (score: 100)
2. **Keyword overlap**: Insight keywords ∩ skill description (score: 60-90)
3. **Technology alignment**: Same framework/library family (score: 40-60)
4. **No match**: Score <40 → Skip enhancement and propose a new skill instead.

### Conflict Detection
Before drafting an enhancement, scan the target file for existing guidance about the same behavior.

| Conflict Type | Action |
|---------------|--------|
| Exact duplicate | Skip quietly |
| Newer source supersedes old rule | Propose a replacement with both source links |
| Context-specific exception | Qualify the existing rule instead of appending a competing rule |
| Equal-confidence contradiction | Ask the user to choose replace, keep both with scopes, or skip |
| Repeated pattern across multiple skills | Propose moving the shared rule to a common skill or reference |

---

## Phase 4: Enhancement Proposal

### For Each Match (score >= 40)
**1. Read current skill:** Read the contents of the matched skill/prompt file.
**2. Identify target section:** Find the best section (e.g., `Patterns`, `Anti-Patterns`, `Quick Reference`).
**3. Draft the enhancement:**
- Preserve the existing structure exactly.
- Add the insight in the appropriate format for that section.
- You MUST include timestamped source attribution: `<!-- Learned: {YYYY-MM-DD} from {url/file} -->`
- Include the behavior change, not just the fact.
- When replacing or qualifying old guidance, show the old rule and new rule in the preview.

### Pruning Signal
If the new insight makes existing guidance stale, generic, or redundant, include a deletion in the same proposal. Learning should improve skill density, not only increase length.

### Health Signal
When reading a target skill, note health issues that affect future pruning:
- Sections with no `Learned:` or source attribution.
- Repeated rules across multiple skills.
- Old learned entries that may have become Tier 1 training data or stale framework advice.
- Skill bodies approaching the size limit where references or splitting would preserve density.

---

## Phase 5: User Approval

### The Proposal Format
For each valid enhancement, you must present the proposal to the user exactly like this:

Present the:
1. Target Skill name
2. Insight summary, Novelty Tier, and Leverage score
3. Placement decision
4. Diff preview of what you are going to add, replace, qualify, or delete
5. Source attribution

**ACTION:** Ask the user: `"Apply this enhancement?"` with the options: `Apply`, `Skip`, or `Edit`.

### Response Handling
- **Apply**: Proceed to Phase 6.
- **Skip**: Skip to the next candidate.
- **Edit**: User modifies the text, then you proceed to Phase 6.

---

## Phase 6: Apply & New Skill Proposal

### 6a. Apply Enhancement
**ACTION:** If the user selected 'Apply', you MUST immediately use your file editing tool to insert the drafted block into the target file. **Do not just say you will do it, execute the tool.**

### 6b. When No Match Found (New Skills)
For insights with no match (score <40), present the user with a summary of the domain and keywords.
Ask: `"Propose new skill for {domain}?" [y/n]`
If approved, generate a structured skill stub. Do not create an empty skill.

Minimum stub:
```markdown
---
name: {domain-slug}
description: {action-oriented trigger and scope boundary}
---

# {Skill Name}

## When to Use

- {specific trigger}

## Workflow

1. {first behavior-changing step}

## Validation

- {how to know the skill worked}

<!-- stub: needs enhancement -->
<!-- Learned: {YYYY-MM-DD} from {url/file} -->
```

---

## Quality Gates

### Absolute Rules
- [ ] Zero Tier 1 insights in skills
- [ ] Zero low-leverage trivia in active skill bodies
- [ ] User approves each change (no auto-apply)
- [ ] Diff preview shown before any edit
- [ ] Timestamped source attribution in comments
- [ ] Conflicts are replaced, qualified, split, or explicitly skipped

## Examples

### Example 1: Extracting from Documentation
**Source**: User runs `/learn https://svelte.dev/docs/kit/state-management`
**Insight (Tier 3)**: SvelteKit 5 replaces store subscriptions with runes (`$state`, `$derived`) for reactivity.
**Target Skill**: `sveltekit-patterns`
**Agent Output Preview**:
```markdown
## Enhancement Proposal (Score: 85, Tier: 3, Leverage: 3)

**Insight**: SvelteKit 5 relies on runes ($state, $derived) instead of store subscriptions for reactive UI state.
**Target Skill**: skills/sveltekit-patterns/SKILL.md
**Section**: Patterns
**Placement**: Existing skill body; changes recurring implementation behavior.

**Proposed Addition**:
### Runes vs Stores (Svelte 5)
<!-- Source: https://svelte.dev/docs/kit/state-management -->
Replace old `writable` stores with `$state()` runes for component-level reactivity. Do not use `$:` for derived state; use `$derived()` instead.

Apply this enhancement? [y/n/edit]
```
