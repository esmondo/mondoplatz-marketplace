# Business Autopilot

A Claude Code plugin that gives you a full team of specialized AI agents to run your business.

## What is this?

Instead of using one general AI assistant, this plugin gives you **department-based agents** where each one handles exactly one job. Think of it as hiring a team, but the team lives in a folder.

## Departments

### Engineering
- `frontend-developer` - Build clean UI from designs or descriptions
- `backend-architect` - Design and implement APIs and data systems

### Product
- `trend-researcher` - Research market trends and opportunities
- `feedback-synthesizer` - Turn user feedback into product insights
- `sprint-prioritizer` - Decide what gets built next

### Marketing
- `content-creator` - Transform ideas into ready-to-post content
- `tiktok-strategist` - Build viral TikTok strategies and content calendars

### Design
- `ui-designer` - Create interface designs with clear hierarchy
- `copywriter` - Write clear, compelling UI copy

### Project Management
- `project-shipper` - Track progress and ensure things ship

### Operations
- `support-responder` - Draft replies to customer inquiries
- `analytics-reporter` - Explain what the numbers mean
- `finance-tracker` - Make the money situation clear

### Testing
- `test-results-analyzer` - Assess release readiness

## Commands

- `/weekly-review` - Run a comprehensive weekly business review
- `/idea-to-content` - Transform an idea into multi-platform content
- `/ship-feature` - End-to-end feature development workflow

## Installation

1. Add the marketplace:
   ```
   /plugin marketplace add esmondo/business-autopilot
   ```

2. Install the plugin:
   ```
   /plugin install business-autopilot@esmondo
   ```

3. Restart Claude Code

4. Check it worked:
   ```
   /agents
   ```

## Usage

**Use a specific agent:**
```
@frontend-developer Build a login form with email and password fields
```

**Use a command:**
```
/idea-to-content I want to post about why most AI agents fail
```

**Let Claude route automatically:**
Just describe what you need, and the workflow routing skill will pick the right agent.

## Customization

Each `.md` file is a prompt you can edit. To customize:

1. Find the agent in `agents/[department]/[agent-name].md`
2. Edit the inputs, process, or output sections
3. Reinstall the plugin to apply changes

## Philosophy

- **One agent, one job**: No "general helper" agents
- **Clear inputs and outputs**: Every agent knows what it needs and what it produces
- **Chainable workflows**: Route work through multiple agents like a real team
- **Start small**: Begin with 3-4 agents, add more as you find bottlenecks

## Adding New Agents

Create a new `.md` file in the appropriate department folder:

```markdown
---
description: One-line description of what this agent does
---

# Agent Name

## Inputs
- What the agent needs to do its job

## Process
1. Step-by-step approach

## Output
- What the agent delivers

## Constraints
- Guardrails and quality standards
```

---

Built for solopreneurs and small teams who want to scale without hiring.
