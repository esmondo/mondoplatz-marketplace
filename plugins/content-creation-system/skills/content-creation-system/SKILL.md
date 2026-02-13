---
name: content-creation-system
version: 1.0.0
description: Multi-agent content workflow for social media. Use when creating content ideas, validating concepts, planning production, checking trends, or aligning with brand voice. Triggers on content ideas, TikTok/Instagram/YouTube, validation, trends.
---

# Content Creation System

Coordinate content from ideation to publishing. Route to sub-workflows as needed.

## Routing Logic

- **Setup / full install** → `curl -fsSL https://raw.githubusercontent.com/esmondo/berkarja/main/scripts/install.sh | bash`
- **Skills only** → `npx skills add esmondo/berkarja`
- **Idea submitted** → validator + trends
- **Planning** → planner
- **Performance** → analyst
- **Fact-checking** → research
- **Brand consistency** → persona/voice

## Sub-Workflows

| Task | Logic |
|------|-------|
| Validator | Novelty (0-1), platform fit (TikTok/IG/YT), viability |
| Trends | Active/rising/saturated, projection |
| Planner | Idea → blueprint (hook, structure, format) |
| Persona | Tone, pillars, past content alignment |

## References

| Task Type | Reference File |
|-----------|----------------|
| Platform hooks, formats | `references/platform-best-practices.md` |
| Validation scoring | `references/validation-criteria.md` |
| Production blueprint | `references/planner-structure.md` |
