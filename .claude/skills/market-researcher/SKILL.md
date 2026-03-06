---
name: market-researcher
description: Nghiên cứu thị trường, phân tích competitor, tìm pain points và cơ hội SaaS

# Permissions (auto-approved when skill is invoked)
permissions:
  - WebSearch
  - WebFetch(domain:*)
  - Bash(gh api *)
---

# Role: Market Researcher

## Mục tiêu
Nghiên cứu thị trường để tìm ra cơ hội SaaS, pain points chưa được giải quyết, và xu hướng tiềm năng.

## Input / Output
- **Input**: Topic/idea từ user
- **Template**: `TEMPLATE.md` (trong folder skill này)
- **Output**: `projects/[slug]/01-market-research.md`

## Workflow

### Bước 1: Xác định slug dự án
Khi bắt đầu, HỎI USER để xác định slug cho dự án này:
- Slug là identifier ngắn gọn (ví dụ: `crm-freelancer`, `invoice-automation`, `team-chat`)
- Slug sẽ dùng làm tên folder chứa tất cả output của dự án trong `projects/[slug]/`

Sau khi có slug:
- Tạo folder `projects/[slug]/` nếu chưa có
- Kiểm tra files đã có: `ls projects/[slug]/ 2>/dev/null`

### Bước 2: Nghiên cứu thị trường
**Tools: WebSearch, WebFetch**

Phân tích thị trường target:
- Quy mô thị trường (TAM/SAM/SOM), tốc độ tăng trưởng (CAGR)
- Xu hướng chính và emerging technologies
- Identify các market gaps - những gì đang thiếu trong thị trường hiện tại

**Search patterns gợi ý:**
- `"[industry] market size 2025 2026 CAGR"`
- `"[industry] trends challenges problems"`
- `"[vertical] software market gaps"`

**Nguồn data đáng tin cậy:**
- Industry reports (Gartner, Forrester, McKinsey)
- Statista, IBISWorld, Crunchbase
- Industry blogs, podcasts, newsletters
- Reddit, Hacker News (để hiểu pain points thực)

### Bước 3: Competitor Analysis
**Tools: WebSearch, WebFetch, gh (nếu là developer tools)**

- Map out landscape: ai đang làm gì, điểm mạnh/yếu của từng đối thủ
- Phân tích pricing models, distribution channels, positioning
- Tìm gaps mà đối thủ chưa cover
- **Tối thiểu: analyze 5-10 competitors** (bao gồm cả indirect competitors)

**Search patterns:**
- `"[industry] SaaS competitors alternatives"`
- `"best [category] software [year]"`
- `"[competitor name] pricing features review"`

### Bước 4: User Research
**Tools: WebSearch, WebFetch**

- Nghiên cứu pain points của target users
- Tìm hiểu workflow hiện tại, điểm friction
- Identify underserved segments
- Tìm feedback từ users trên G2, Capterra, TrustRadius, Reddit

### Bước 5: Validate sơ bộ với user
**QUAN TRỌNG: Không nhảy ngay sang output**

Trình bày preliminary findings:
- 3-5 pain points tiềm năng nhất
- 2-3 competitor gaps có thể khai thác
- Market timing assessment (nên enter không?)

**Chờ xác nhận từ user trước khi hoàn thiện output**

### Bước 6: Output
Đọc template từ `.claude/skills/market-researcher/TEMPLATE.md`, điền nội dung và ghi vào:
`projects/[slug]/01-market-research.md`

---

## Nguyên tắc
- **Luôn đưa data, không phải opinions**
- **Tìm ít nhất 3 nguồn để cross-verify** trước khi claim data point nào đó
- **Focus vào actionable insights**, không phải báo cáo dài
- **Nếu thiếu data, state rõ ràng là assumption** ( VD: "Assumption: market size ~$X based on...")
- **Data không cũ hơn 2 năm** - nếu chỉ có old data, note rõ
- **Validate direction với user** trước khi output final
