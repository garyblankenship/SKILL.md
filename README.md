<p align="center">
  <img src="gnome-banner.jpg" alt="Skills & Crafts AI Programmers Guild" width="400">
</p>

# SKILL.md — Self-Improving Claude Code

**What if your AI assistant could teach itself?**

That's the idea behind this repo. Most Claude Code setups have static prompts that rot over time. New framework version? Your skills are stale. Better pattern discovered? Manual updates. Documentation changed? You're behind.

`/learn` fixes this. Point it at any source—URL, file, skill marketplace, or codebase—and watch it extract patterns, match them to your existing skills, and propose targeted enhancements. Your Claude Code setup becomes self-improving.

```bash
/learn https://svelte.dev/docs/kit          # Learn from documentation
/learn ~/docs/architecture.md               # Learn from local files
/learn ~/.claude/plugins/marketplaces/      # Copy skills from marketplaces
/learn ~/projects/my-api                    # Extract patterns from codebases
```

Read the SvelteKit 5 migration guide? Your `sveltekit-patterns` skill now knows about runes. Found a great Express middleware pattern? Your `api-middleware` skill just got smarter. Discovered a skill marketplace? Copy the good ones directly.

---

## How It Works

Three layers, each doing one thing well:

```
/learn <source>
    ↓
┌─────────────────────────────────────────────────────────────┐
│  COMMAND (26 lines)                                         │
│  Thin trigger. Delegates everything to the specialist.      │
└─────────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────────┐
│  AGENT (skill-learning-specialist)                          │
│  The orchestrator. Handles URLs, directories, repos.        │
│  Manages the approval loop. Doesn't contain methodology.    │
└─────────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────────┐
│  SKILL (skill-learning)                                     │
│  The brain. 7-phase workflow. Novelty detection.            │
│  Matching algorithm. CLEAR validation. All the methodology. │
└─────────────────────────────────────────────────────────────┘
```

**The pattern**: Commands trigger. Agents orchestrate. Skills contain methodology.

This separation matters. Commands stay tiny. Agents handle real-world messiness (blocked URLs, directory structures, user approval). Skills hold reusable knowledge that multiple agents can load.

---

## What `/learn` Can Do

### Learn from URLs
```bash
/learn https://docs.example.com/guide
```
Fetches the page (with Jina fallback for blocked sites), extracts technical patterns, filters out training data using novelty detection, matches insights to your skills, shows diffs, applies on approval.

### Learn from Files
```bash
/learn ~/docs/architecture.md
```
Same pipeline, local source. Great for internal documentation, design docs, or notes you want to codify into skills.

### Learn from Skill Marketplaces
```bash
/learn ~/.claude/plugins/marketplaces/my-skills/
```
Discovers skills via AGENTS.md manifests or glob patterns. You choose: copy skills wholesale, or extract insights to enhance existing skills.

### Learn from Codebases
```bash
/learn ~/projects/my-api
```
Detects project type (Node.js, Go, Rust, Python). Finds validators, schemas, middleware, presets. Extracts patterns. Creates NEW skills from what it finds.

---

## The Secret Sauce: Novelty Detection

Not everything in documentation is worth learning. `/learn` uses a 4-tier novelty framework:

| Tier | Include? | What It Means |
|------|----------|---------------|
| 1 | **EXCLUDE** | Training data—Claude already knows this |
| 2 | Include | Implementation-specific—shows HOW |
| 3 | High value | Architectural trade-offs—explains WHY |
| 4 | Highest | Counter-intuitive—contradicts assumptions |

Only Tier 2-4 insights make it into your skills. No bloat. No redundancy. Just the stuff that actually makes your setup better.

---

## Repository Structure

```
SKILL.md/
├── README.md                      # You are here
├── gnome-banner.jpg               # Our distinguished mascot
└── examples/
    └── learn/                     # The /learn command example
        ├── README.md              # Overview and installation
        ├── commands/
        │   └── learn.md           # The slash command (copy to ~/.claude/commands/)
        ├── agents/
        │   └── skill-learning-specialist.md   # The agent (copy to ~/.claude/agents/)
        └── skills/
            └── skill-learning/
                └── SKILL.md       # The skill (copy to ~/.claude/skills/skill-learning/)
```

Each example mirrors the `~/.claude/` directory structure. Copy directly to your config.

---

## Using These Examples

**Option 1: Copy directly**

Each example mirrors `~/.claude/` structure. Just copy:

```bash
# Install the /learn example
cp -r examples/learn/commands/* ~/.claude/commands/
cp -r examples/learn/agents/* ~/.claude/agents/
cp -r examples/learn/skills/* ~/.claude/skills/
```

**Option 2: Study the patterns**

Read through `examples/learn/README.md`. Understand the architecture. Apply the patterns to your own commands/agents/skills.

**Option 3: Learn from this repo**

Meta, right? Point `/learn` at this repo:
```bash
/learn ~/path/to/SKILL.md/examples/learn/
```

---

## The Architecture Pattern

Every example in this repo follows the same pattern:

1. **Thin commands** — YAML frontmatter + delegation. No logic.
2. **Orchestrating agents** — Handle I/O, errors, user interaction. Load skills for methodology.
3. **Fat skills** — Complete workflows. Reusable across agents. The actual knowledge.

This isn't just organization. It's a forcing function for clean separation of concerns. Commands can't bloat because they only delegate. Agents can't hardcode methodology because it lives in skills. Skills stay focused because they're loaded on demand.

---

## Validation

Lint your components before committing with [cclint](https://github.com/dotcommander/cclint):

```bash
# Install
go install github.com/dotcommander/cclint@latest

# Lint a file
cclint ~/.claude/agents/my-agent.md -v

# Lint all components
cclint ~/.claude/
```

Catches schema violations, bloated agents, missing sections, and broken references. The examples in this repo pass `cclint` validation.

---

## Contributing

Found a pattern worth sharing? Built a useful command chain?

1. Fork this repo
2. Add your example to `examples/`
3. Follow the format: hook-first intro, clear value proposition, full source code
4. Run `cclint examples/your-example/ --type <agent|command|skill>` to validate
5. PR it

Keep examples generic and reusable. Specific tool names are fine. Paths to your personal directories are not.

---

## Credits

- [cclint](https://github.com/dotcommander/cclint) — Claude Code component linter by [@dotcommander](https://github.com/dotcommander)

---

## License

MIT. Use it, remix it, improve it. If something breaks, file an issue. If something works brilliantly, also file an issue—we like hearing about wins.

---

<p align="center">
  <i>Static prompts rot. Self-improving configurations don't.</i>
</p>
