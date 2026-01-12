# Workflow Routing Skill

This skill enables intelligent routing of tasks to the appropriate department agents.

## Purpose

When a user describes a task or problem without specifying which agent to use, analyze the request and route it to the correct agent(s).

## Routing Logic

### Engineering Department
Route here when the request involves:
- Building UI components or screens → `frontend-developer`
- API design, database work, server logic → `backend-architect`
- Technical architecture decisions → `backend-architect`

### Product Department
Route here when the request involves:
- Market research, competitive analysis → `trend-researcher`
- User feedback analysis, feature requests → `feedback-synthesizer`
- Deciding what to build next, prioritization → `sprint-prioritizer`

### Marketing Department
Route here when the request involves:
- Creating posts, articles, or social content → `content-creator`
- Platform-specific strategy → use platform-specific agent if available

### Design Department
Route here when the request involves:
- Visual design, layouts, UI patterns → `ui-designer`
- Interface text, microcopy, error messages → `copywriter`

### Project Management
Route here when the request involves:
- Tracking progress, shipping deadlines → `project-shipper`
- Status updates, blocker identification → `project-shipper`

### Operations
Route here when the request involves:
- Customer support drafts → `support-responder`
- Data analysis, metrics interpretation → `analytics-reporter`
- Financial tracking, cash flow → `finance-tracker`

### Testing
Route here when the request involves:
- QA results, bug analysis → `test-results-analyzer`
- Release readiness assessment → `test-results-analyzer`

## Multi-Agent Workflows

Some requests require chaining multiple agents:

**Idea → Content**
1. `trend-researcher` (validate angle)
2. `content-creator` (produce content)

**Feature Development**
1. `feedback-synthesizer` (validate need)
2. `sprint-prioritizer` (schedule it)
3. `ui-designer` + `copywriter` (design it)
4. `frontend-developer` + `backend-architect` (build it)
5. `test-results-analyzer` (verify it)

**Weekly Review**
1. `analytics-reporter` (numbers)
2. `feedback-synthesizer` (user sentiment)
3. `finance-tracker` (cash position)
4. `project-shipper` (delivery status)

## Routing Behavior

1. If the request clearly maps to one agent, use that agent directly
2. If the request spans multiple domains, identify the workflow and run agents in sequence
3. If unclear, ask one clarifying question to determine the right routing
4. Always explain which agent(s) you're using and why
