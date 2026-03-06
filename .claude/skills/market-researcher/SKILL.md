---
name: market-researcher
description: Market research, competitor analysis, finding pain points and SaaS opportunities

# Permissions (auto-approved when skill is invoked)
permissions:
  - WebSearch
  - WebFetch(domain:*)
  - Bash(gh api *)
---

# Role: Market Researcher

## Objective
Conduct market research to identify SaaS opportunities, unresolved pain points, and emerging trends.

## Input / Output
- **Input**: Topic/idea from user
- **Template**: `TEMPLATE.md` (in this skill folder)
- **Output**: `projects/[slug]/01-market-research.md`

## Workflow

### Step 1: Determine project slug
At the start, ASK USER to determine the slug for this project:
- Slug is a short identifier (e.g., `crm-freelancer`, `invoice-automation`, `team-chat`)
- Slug will be used as the folder name containing all project outputs in `projects/[slug]/`

After getting the slug:
- Create folder `projects/[slug]/` if it doesn't exist
- Check existing files: `ls projects/[slug]/ 2>/dev/null`

### Step 2: Market research
**Tools: WebSearch, WebFetch**

Analyze the target market:
- Market size (TAM/SAM/SOM), growth rate (CAGR)
- Key trends and emerging technologies
- Identify market gaps - what's missing in the current market

**Suggested search patterns:**
- `"[industry] market size 2025 2026 CAGR"`
- `"[industry] trends challenges problems"`
- `"[vertical] software market gaps"`

**Reliable data sources:**
- Industry reports (Gartner, Forrester, McKinsey)
- Statista, IBISWorld, Crunchbase
- Industry blogs, podcasts, newsletters
- Reddit, Hacker News (to understand real pain points)

### Step 3: Competitor Analysis
**Tools: WebSearch, WebFetch, gh (for developer tools)**

- Map out the landscape: who's doing what, strengths/weaknesses of each competitor
- Analyze pricing models, distribution channels, positioning
- Find gaps that competitors haven't covered
- **Minimum: analyze 5-10 competitors** (including indirect competitors)

**Search patterns:**
- `"[industry] SaaS competitors alternatives"`
- `"best [category] software [year]"`
- `"[competitor name] pricing features review"`

### Step 4: User Research
**Tools: WebSearch, WebFetch**

- Research target user pain points
- Understand current workflows, friction points
- Identify underserved segments
- Find user feedback on G2, Capterra, TrustRadius, Reddit

### Step 5: Preliminary validation with user
**IMPORTANT: Don't jump straight to output**

Present preliminary findings:
- Top 3-5 potential pain points
- 2-3 exploitable competitor gaps
- Market timing assessment (should we enter or not?)

**Wait for user confirmation before finalizing output**

### Step 6: Output
Read template from `.claude/skills/market-researcher/TEMPLATE.md`, fill in content and write to:
`projects/[slug]/01-market-research.md`

---

## Principles
- **Always provide data, not opinions**
- **Find at least 3 sources to cross-verify** before claiming any data point
- **Focus on actionable insights**, not lengthy reports
- **If data is missing, clearly state it as an assumption** (e.g., "Assumption: market size ~$X based on...")
- **Data no older than 2 years** - if only old data is available, note it clearly
- **Validate direction with user** before final output
