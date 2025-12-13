# The /learn Command

**What if Claude could teach itself?**

That's what `/learn` does. Point it at a URL, file, skill marketplace, or entire codebase—and watch it extract patterns, match them to your existing skills, and propose targeted enhancements. Your Claude Code setup becomes self-improving.

```bash
/learn https://svelte.dev/docs/kit          # Learn from documentation
/learn ~/docs/architecture.md               # Learn from local files
/learn ~/.claude/plugins/marketplaces/      # Copy skills from marketplaces
/learn ~/projects/my-api                    # Extract patterns from codebases
```

---

## The Chain

Three layers, each doing one thing well:

```
/learn <source>
    ↓
┌─────────────────────────────────────────────────────────────┐
│  commands/learn.md                                          │
│  Thin trigger. Delegates everything to the specialist.      │
└─────────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────────┐
│  agents/skill-learning-specialist.md                        │
│  The orchestrator. Handles URLs, directories, repos.        │
│  Manages the approval loop. Loads skill for methodology.    │
└─────────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────────┐
│  skills/skill-learning/SKILL.md                             │
│  The brain. 7-phase workflow. Novelty detection.            │
│  Matching algorithm. CLEAR validation. All the methodology. │
└─────────────────────────────────────────────────────────────┘
```

**The pattern**: Commands trigger. Agents orchestrate. Skills contain methodology.

---

## Structure

```
learn/
├── README.md                              # You are here
├── commands/
│   └── learn.md                           # The slash command
├── agents/
│   └── skill-learning-specialist.md       # The orchestrating agent
└── skills/
    └── skill-learning/
        └── SKILL.md                       # The methodology skill
```

---

## Installation

Copy to your Claude Code config:

```bash
# Command
cp commands/learn.md ~/.claude/commands/

# Agent
cp agents/skill-learning-specialist.md ~/.claude/agents/

# Skill
mkdir -p ~/.claude/skills/skill-learning
cp skills/skill-learning/SKILL.md ~/.claude/skills/skill-learning/
```

---

## Why This Matters

Most Claude Code setups have static prompts that rot over time. `/learn` makes your configuration self-improving:

- Read the SvelteKit 5 migration guide? Your `sveltekit-patterns` skill now knows about runes.
- Found a plugin marketplace? Copy the good skills directly.
- Analyzing a reference implementation? Extract the patterns into new skills.

The secret sauce is **novelty detection**—only Tier 2-4 insights make it into your skills. No training data bloat. Just the stuff that actually makes your setup better.
