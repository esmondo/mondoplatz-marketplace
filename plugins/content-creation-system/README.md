# Content Creation System Plugin for Claude Code

Multi-agent content creation system with idea validation, trend intelligence, and production planning for social media.

## Features

- **Idea validation** — Novelty, platform fit, viability scoring
- **Trend intelligence** — Active/rising/saturated trends, projection
- **Production planning** — Blueprints, hooks, repurposing
- **Brand persona** — Consistency checks against voice and pillars

## Installation

1. Add the marketplace (if not already added):
   ```
   /plugin marketplace add esmondo/mondoplatz
   ```

2. Install the plugin:
   ```
   /plugin install content-creation-system@mondo
   ```

3. Restart Claude Code

## Full Setup (optional)

**Skills only** (any directory):
```bash
npx skills add esmondo/berkarja
```

**Full install** (Python, setup wizard):
```bash
curl -fsSL https://raw.githubusercontent.com/esmondo/berkarja/main/scripts/install.sh | bash
```

---

Built by Mondo for content creators.
