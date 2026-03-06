# OpenCompany

A framework for developing SaaS products from idea to launch, built on Claude Code.

## Overview

OpenCompany is a SaaS incubator that helps entrepreneurs and product teams systematically develop SaaS products through 6 specialized AI roles. Each role handles a specific phase of the product lifecycle, from market research to post-launch operations.

## 6 AI Roles

| Role | Responsibility |
|------|----------------|
| **Market Researcher** | Market analysis, competitor research, pain point and opportunity identification |
| **Product Strategist** | Idea validation, MVP definition, positioning and business model |
| **Technical Architect** | Architecture design, tech stack selection, system design from MVP to scale |
| **Implementation Lead** | Development from setup to deploy, ensuring quality and velocity |
| **Growth Strategist** | Go-to-market strategy, acquisition and retention |
| **SaaS Operator** | Post-launch operations, optimization and scaling |

## Directory Structure

```
├── skills/                   # 6 AI roles
│   ├── market-researcher/
│   ├── product-strategist/
│   ├── technical-architect/
│   ├── implementation-lead/
│   ├── growth-strategist/
│   └── saas-operator/
├── projects/                 # SaaS projects
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

## Usage

### Workflow

Each SaaS project goes through 6 sequential phases:

```
Market Research → Product Strategy → Technical Architecture
    → Implementation → Growth Strategy → Operations
```

Each phase reads the output of the previous phase and produces deliverables for the next.

### Invoking Roles

```
/market-researcher     # Market research
/product-strategist    # Product strategy
/technical-architect   # Technical architecture
/implementation-lead   # Implementation
/growth-strategist     # Growth strategy
/saas-operator         # Operations
```

### Process

1. Create a new project directory under `projects/`
2. Run `/market-researcher` to conduct market research
3. Review and validate findings
4. Run the next role — each role automatically reads prior output
5. Repeat until all 6 phases are complete

## Principles

- **Data-driven** — Every decision backed by data
- **Kill ideas early** — Discard non-viable ideas fast
- **Validate with users** — Confirm with users before finalizing
- **MVP first** — Focus on the minimum viable feature set
- **Measure everything** — Track all metrics post-launch

## Projects

| Project | Description | Status |
|---------|-------------|--------|
| **AI Chat App** | AI-native team collaboration tool with task delegation for SMBs | Market research done |
| **Note App AI** | Markdown note-taking app with AI assistance for developers | Market research done |

## Requirements

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- Git

## License

MIT
