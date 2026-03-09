# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is **OpenCompany**, a SaaS incubator framework that guides users through building SaaS products from idea to launch using 6 specialized AI roles. Each role is invoked as a Claude Code skill and produces deliverables that feed into the next phase.

## Project Workflow

Projects progress through 6 sequential phases:

```
Market Research → Product Strategy → Technical Architecture
    → Implementation → Growth Strategy → Operations
```

## Invoking AI Roles (Skills)

Use the `/` slash commands to invoke specialized AI roles:

| Command | Phase | Output |
|---------|-------|--------|
| `/market-researcher` | Phase 1: Market Research | `projects/[slug]/01-market-research.md` |
| `/product-strategist` | Phase 2: Product Strategy | `projects/[slug]/02-product-strategy.md` |
| `/technical-architect` | Phase 3: Technical Architecture | `projects/[slug]/03-technical-architecture.md` |
| `/implementation-lead` | Phase 4: Implementation | `projects/[slug]/04-implementation.md` |
| `/growth-strategist` | Phase 5: Growth Strategy | `projects/[slug]/05-growth-strategy.md` |
| `/saas-operator` | Phase 6: Operations | `projects/[slug]/06-operations.md` |

## Directory Structure

```
projects/                          # SaaS project outputs
  └── [project-slug]/              # Each project has its own folder
      ├── 01-market-research.md   # Phase 1 output
      ├── 02-product-strategy.md  # Phase 2 output
      ├── 03-technical-architecture.md  # Phase 3 output
      ├── 04-implementation.md    # Phase 4 output
      ├── 05-growth-strategy.md  # Phase 5 output
      └── 06-operations.md       # Phase 6 output
.claude/skills/                   # AI role skills (6 roles)
  └── [role-name]/
      ├── SKILL.md              # Role definition and workflow
      └── TEMPLATE.md            # Output template
```

## Creating a New Project

1. Create a new project folder: `mkdir projects/[slug]/`
2. Run `/market-researcher` with the project idea
3. After each phase completes, run the next role — each role automatically reads the prior phase's output

## Key Principles

- **Data-driven**: Every decision backed by data (verify sources)
- **Kill ideas early**: Discard non-viable ideas fast
- **Validate with users**: Confirm with real users before finalizing
- **MVP first**: Focus on minimum viable feature set
- **Measure everything**: Track all metrics post-launch
- **Validate direction**: Present preliminary findings to user before finalizing output

## Skill Workflow Pattern

Each skill follows this pattern:
1. Ask user for project slug
2. Check that prior phase output exists
3. Review prior output
4. Research/analyze using WebSearch, WebFetch
5. Present preliminary findings to user
6. Wait for user confirmation
7. Read template from `.claude/skills/[role]/TEMPLATE.md`
8. Write output to `projects/[slug]/[phase-number]-[phase-name].md`

## Permissions

Skills have different permissions:
- **market-researcher**: WebSearch, WebFetch, gh api
- **product-strategist**: WebSearch, WebFetch
- **technical-architect**: WebSearch, WebFetch
- **implementation-lead**: WebSearch, WebFetch, git, npm, npx
- **growth-strategist**: WebSearch, WebFetch
- **saas-operator**: WebSearch, WebFetch, gh
