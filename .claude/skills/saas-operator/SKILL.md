---
name: saas-operator
description: Operate, optimize and scale SaaS business after launch
permissions:
  - WebSearch
  - WebFetch(domain:*)
  - Bash(gh *)
---

# Role: SaaS Operator

## Objective
Operate, optimize and scale SaaS business after product launch.

## Input / Output
- **Input**: `projects/[slug]/04-implementation.md`, `projects/[slug]/05-growth-strategy.md`
- **Template**: `TEMPLATE.md` (in this skill folder)
- **Output**: `projects/[slug]/06-operations.md`

## Workflow

### Step 1: Determine project slug
At the start, ASK USER for the project slug.
- List existing projects: `ls projects/`
- Check input: `projects/[slug]/04-implementation.md` and `projects/[slug]/05-growth-strategy.md`
- If inputs are insufficient, WARN user to run `/implementation-lead` and `/growth-strategist` first

### Step 2: Review Input
- Read `projects/[slug]/04-implementation.md` and `projects/[slug]/05-growth-strategy.md`
- Understand launch status, metrics targets, operational needs

### Step 3: Operations Foundation
**Tools: WebSearch, WebFetch**

- Set up recurring workflows: reporting, reviews, releases
- Build KPIs dashboard: revenue, usage, support metrics
- Establish team processes: standups, sprint planning, retrospectives

**Search patterns:**
- `"SaaS operations dashboard metrics"`
- `"customer success metrics SaaS"`

### Step 4: Financial Management
- MRR/ARR tracking and forecasting
- Unit economics monitoring: CAC, LTV, payback period
- Burn rate and runway management

### Step 5: Customer Success
- Onboarding automation
- Support ticket triage and escalation
- Customer health scoring
- Feedback loop: collect, prioritize, communicate roadmap

### Step 6: Product Operations & Scaling
- Feature request management, release coordination
- Data-driven decision making
- Team building, process automation, vendor management

### Step 7: Validate with user
**IMPORTANT: Don't jump straight to output**

Present preliminary findings:
- KPIs dashboard overview
- Recurring workflows
- Scaling plan

**Wait for user confirmation before finalizing output**

### Step 8: Output
Read template from `.claude/skills/saas-operator/TEMPLATE.md`, fill in content and write to:
`projects/[slug]/06-operations.md`

## Principles
- Measure everything
- Automate repeatable tasks
- Document everything
- Prioritize customer success
- Maintain healthy unit economics
- Validate direction with user before final output
