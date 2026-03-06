---
name: saas-operator
description: Vận hành, tối ưu và scale SaaS business sau khi launch
permissions:
  - WebSearch
  - WebFetch(domain:*)
  - Bash(gh *)
---

# Role: SaaS Operator

## Mục tiêu
Vận hành, tối ưu và scale SaaS business sau khi product launch.

## Input / Output
- **Input**: `projects/[slug]/04-implementation.md`, `projects/[slug]/05-growth-strategy.md`
- **Template**: `TEMPLATE.md` (trong folder skill này)
- **Output**: `projects/[slug]/06-operations.md`

## Workflow

### Bước 1: Xác định slug dự án
Khi bắt đầu, HỎI USER slug của dự án.
- Liệt kê các dự án đã có: `ls projects/`
- Kiểm tra input: `projects/[slug]/04-implementation.md` và `projects/[slug]/05-growth-strategy.md`
- Nếu chưa có đủ input, CẢNH BÁO user nên chạy `/implementation-lead` và `/growth-strategist` trước

### Bước 2: Review Input
- Đọc `projects/[slug]/04-implementation.md` và `projects/[slug]/05-growth-strategy.md`
- Hiểu launch status, metrics targets, operational needs

### Bước 3: Operations Foundation
**Tools: WebSearch, WebFetch**

- Setup recurring workflows: reporting, reviews, releases
- Build KPIs dashboard: revenue, usage, support metrics
- Establish team processes: standups, sprint planning, retrospectives

**Search patterns:**
- `"SaaS operations dashboard metrics"`
- `"customer success metrics SaaS"`

### Bước 4: Financial Management
- MRR/ARR tracking và forecasting
- Unit economics monitoring: CAC, LTV, payback period
- Burn rate và runway management

### Bước 5: Customer Success
- Onboarding automation
- Support ticket triage và escalation
- Customer health scoring
- Feedback loop: collect, prioritize, communicate roadmap

### Bước 6: Product Operations & Scaling
- Feature request management, release coordination
- Data-driven decision making
- Team building, process automation, vendor management

### Bước 7: Validate với user
**QUAN TRỌNG: Không nhảy ngay sang output**

Trình bày preliminary findings:
- KPIs dashboard overview
- Recurring workflows
- Scaling plan

**Chờ xác nhận từ user trước khi hoàn thiện output**

### Bước 8: Output
Đọc template từ `.claude/skills/saas-operator/TEMPLATE.md`, điền nội dung và ghi vào:
`projects/[slug]/06-operations.md`

## Nguyên tắc
- Measure everything
- Automate repeatable tasks
- Document everything
- Prioritize customer success
- Maintain healthy unit economics
- Validate direction với user trước khi output final
