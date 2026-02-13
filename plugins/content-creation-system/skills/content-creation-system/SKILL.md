---
name: content-creation-system
version: 1.0.0
description: Multi-agent content workflow for social media. Use when creating content ideas, validating concepts, planning production, checking trends, or aligning with brand voice. Triggers on content ideas, TikTok/Instagram/YouTube, validation, trends.
---

# Content Creation System

Multi-agent workflow for social media content from ideation to publishing. Coordinates validation, trend intelligence, planning, and distribution.

**Full install:** `npx skills add esmondo/berkarja`

---

## When to Apply

Reference this skill when the user:

- Submits a content idea and wants validation (novelty, platform fit)
- Asks "is this idea worth making?" or "will this perform on TikTok/IG/YT?"
- Wants a production plan or blueprint for a validated idea
- Asks about trends (what's active, rising, saturated)
- Needs brand voice alignment check on a draft
- Wants to repurpose content across platforms
- Asks about performance of past content

---

## Sub-Workflows by Priority

| Priority | Workflow   | When                     | Load Reference                  |
|----------|------------|---------------------------|---------------------------------|
| 1        | Validator  | Idea submitted            | `references/validation-criteria.md` |
| 2        | Trends     | Need trend context        | —                               |
| 3        | Planner    | Validated idea → blueprint| `references/planner-structure.md`|
| 4        | Persona    | Brand consistency check   | —                               |
| 5        | Analyst    | Post-publish metrics      | —                               |
| 6        | Distributor| Scheduling, repurposing   | —                               |
| 7        | Research   | Fact-checking, citations  | —                               |

---

## Quick Reference

### Validator Output Format
```json
{
  "novelty": 0.75,
  "fit": { "tiktok": 0.9, "ig": 0.7, "yt": 0.5 },
  "viable": true,
  "rec": ["Strong TikTok hook", "Add visual B-roll for IG"]
}
```

### Platform Fit Criteria
- **TikTok**: Hook in 3s, trend-ready, audio-driven
- **IG**: Visual quality, 15–90s, editing complexity
- **YT**: Depth potential, 8–15min, retention structure

### Blueprint Structure (Planner)
```yaml
concept: [1-line summary]
platform_primary: [TikTok/IG/YT]
format: [talking head/B-roll/screen record]
hook: [First 3s script]
structure: [intro/body/CTA]
repurpose: [vertical→horizontal adaptations]
```

---

## How to Use

### Step 1: Identify the Request

| User says…                    | Route to          |
|------------------------------|-------------------|
| "I have an idea: …"          | Validator + Trends |
| "Plan this for production"   | Planner           |
| "Check if this fits my brand"| Persona           |
| "How did my last video do?"  | Analyst           |

### Step 2: Load References as Needed

| Task               | Reference File                          |
|--------------------|-----------------------------------------|
| Platform rules     | `references/platform-best-practices.md`  |
| Validation scoring | `references/validation-criteria.md`     |
| Blueprint template | `references/planner-structure.md`       |

### Step 3: Full Setup (Optional)

For Python setup wizard and config:
```bash
curl -fsSL https://raw.githubusercontent.com/esmondo/berkarja/main/scripts/install.sh | bash
```

---

## Example Flow

**User:** "I want to make a video about EEG signal artifacts in meditation research"

1. **Validator**: Score novelty (is this overdone?), platform fit (TikTok vs YT depth), viability
2. **Trends**: Check if "meditation tech" or "neurotech" is active/saturated
3. **Planner**: If validated → generate blueprint (hook, structure, format)
4. **Persona**: If user has brand_voice → check alignment before final script

---

## Validation Dimensions (Weights)

| Dimension  | Weight | Description                          |
|------------|--------|--------------------------------------|
| Relevance  | 0.25   | Audience needs, interests            |
| Uniqueness | 0.20   | Differentiation from existing       |
| Feasibility| 0.20   | Resources, time, technical           |
| Alignment  | 0.20   | Brand voice, pillars                 |
| Impact     | 0.15   | Reach, engagement potential          |
