# OpenCompany

A framework for building SaaS products from idea to launch, powered by Claude Code.

## Overview

OpenCompany is a SaaS incubator that guides you through 6 sequential phases, each powered by a specialized AI role. Each role builds on the previous phase's output to systematically develop your SaaS from market research through post-launch operations.

## The 6 AI Roles

| Role | Responsibility |
|------|----------------|
| **Market Researcher** | Analyze markets, research competitors, identify pain points and opportunities |
| **Product Strategist** | Validate ideas, define MVP, establish positioning and business model |
| **Technical Architect** | Design architecture, select tech stack, plan system from MVP to scale |
| **Implementation Lead** | Lead development from setup to deployment, ensuring quality and velocity |
| **Growth Strategist** | Build go-to-market strategy, drive acquisition and retention |
| **SaaS Operator** | Manage post-launch operations, optimize and scale the business |

## Directory Structure

```
├── skills/                      # 6 AI role skills
│   ├── market-researcher/
│   ├── product-strategist/
│   ├── technical-architect/
│   ├── implementation-lead/
│   ├── growth-strategist/
│   └── saas-operator/
├── projects/                    # Your SaaS projects
│   └── [project-slug]/
│       ├── 01-market-research.md
│       ├── 02-product-strategy.md
│       ├── 03-technical-architecture.md
│       ├── 04-implementation.md
│       ├── 05-growth-strategy.md
│       └── 06-operations.md
└── .claude/
    └── settings.local.json
```

## Workflow

Each project progresses through 6 sequential phases:

```
Market Research → Product Strategy → Technical Architecture
    → Implementation → Growth Strategy → Operations
```

Each phase reads the previous phase's output and produces deliverables for the next.

## Usage

### Invoke Roles

```
/market-researcher     # Conduct market research
/product-strategist    # Define product strategy
/technical-architect   # Design technical architecture
/implementation-lead   # Lead implementation
/growth-strategist     # Plan growth strategy
/saas-operator        # Manage operations
```

### Process

1. Create a new project directory under `projects/`
2. Run `/market-researcher` to analyze the market
3. Review and validate findings
4. Run the next role — each role automatically reads prior output
5. Repeat until all 6 phases are complete

## Core Principles

- **Data-driven** — Every decision backed by data
- **Kill ideas early** — Discard non-viable ideas fast
- **Validate with users** — Confirm with real users before finalizing
- **MVP first** — Focus on the minimum viable feature set
- **Measure everything** — Track all metrics post-launch

## Current Projects

| Project | Description | Status |
|---------|-------------|--------|
| **AI Chat App** | AI-native team collaboration tool with task delegation for SMBs | Phase 1: Market Research ✓ |
| **Note App AI** | Markdown note-taking app with AI assistance for developers | Phase 1: Market Research ✓ |
| **HTTP Traffic Monitor** | Real-time HTTP traffic analysis and monitoring tool | Phase 5: Growth Strategy ✓ |

## Requirements

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- Git

## License

MIT
