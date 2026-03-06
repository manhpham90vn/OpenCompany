# HTTP Traffic Monitor & Ad Blocker - Product Strategy

> Date: 2026-03-06
> Input: `../01-market-research.md`
> Status: draft

## Problem Statement

Solo developer và QA engineer mất 10-15 phút setup công cụ debug HTTP, sau đó phải vất vả tìm request cần thiết trong đống quảng cáo/analytics noise — vì không có tool miễn phí nào kết hợp traffic inspection với ad-blocking trong một GUI đơn giản, cross-platform.

## Solution Overview

**FlowProxy** là một cross-platform desktop app giúp developer debug HTTP/HTTPS traffic nhanh hơn bằng cách kết hợp local proxy, traffic inspector, và ad-blocker trong một công cụ duy nhất — miễn phí, không cần setup phức tạp.

## Validation

| Criteria | Assessment | Evidence | Notes |
|----------|-----------|----------|-------|
| Problem (real?) | HIGH | 4M+ devs dùng Fiddler, mitmproxy phổ biến dù UX kém | Developer thực sự đang chịu đựng tool cũ hoặc trả $90+/năm |
| Solution (better?) | HIGH | Không có tool miễn phí nào kết hợp ad-blocking + inspection + GUI đẹp | Gap rõ ràng, chưa ai lấp |
| Market (big enough?) | MEDIUM | SAM ~$500M, SOM ~$10-50M | Đủ lớn cho indie product, không phải VC-scale |
| Timing (why now?) | HIGH | Electron/Tauri trưởng thành, HTTP/3 tạo cơ hội, emerging markets tăng trưởng | Stack để build đã sẵn sàng |

## ICP (Ideal Customer Profile)

### Primary ICP
- Who: Solo developer / freelancer làm web hoặc mobile app
- Industry: Software development, mobile development
- Company size: Individual hoặc startup 1-10 người
- Budget range: $0-20/năm
- Willingness-to-pay: $0 (free), $49/năm (pro)
- Current alternatives: Charles Proxy (đắt), mitmproxy (khó dùng), browser DevTools (giới hạn)

### User Segments

| Segment | Size | Priority | Key Needs |
|---------|------|----------|-----------|
| Solo Developer / Freelancer | ~1M (emerging markets) | P0 | Debug nhanh, miễn phí, không setup phức tạp |
| QA Engineer | ~500K | P1 | Capture + filter requests, giảm noise từ ads |
| Privacy-conscious user | ~2M | P2 | Biết app đang gửi gì, block tracker |

## Value Proposition

### Positioning Statement
> For solo developers và QA engineers, FlowProxy là HTTP debugging tool miễn phí kết hợp traffic inspection và ad-blocking trong một app cross-platform. Unlike Charles/Proxyman (đắt tiền) và mitmproxy (CLI-only), chúng tôi cung cấp GUI hiện đại với one-click setup — không mất tiền, không mất thời gian.

### Key Benefits
1. **Miễn phí và open-source** — không cần trả $90+/năm cho Charles hay Fiddler
2. **Zero noise debugging** — tự động filter ad/tracker traffic, chỉ thấy request quan trọng
3. **One-click setup** — tự động install SSL certificate, không cần config thủ công

### Differentiation
- Tại sao không dùng **mitmproxy**? CLI-only, barrier to entry cao, không có ad-blocking tích hợp
- Tại sao không dùng **Charles/Proxyman**? $50-90 one-time hoặc $90/năm — quá đắt cho individual dev ở emerging markets
- Tại sao không tự build? Setup mitmproxy + uBlock rules mất vài giờ, không có unified UI

## MVP Scope

### Must-have (P0)
1. **Local HTTP/HTTPS proxy** — intercept traffic từ mọi app trên máy, không chỉ browser
2. **One-click SSL certificate setup** — tự động generate và install CA certificate
3. **Traffic inspector UI** — list requests theo thời gian thực, filter, search
4. **Built-in ad/tracker blocking** — tích hợp EasyList + uBlock filter lists

### Nice-to-have (P1)
1. **Request/response detail view** — xem headers, body, timing, size
2. **Quick toggle "hide ad traffic"** — ẩn/hiện ad requests trong cùng 1 view
3. **Export session** — lưu traffic log ra file để share hoặc replay

### Out of Scope (for MVP)
1. Team collaboration / cloud sync
2. Request modification / mocking (rule engine)
3. Mobile companion app (iOS/Android)
4. Plugin ecosystem
5. WebSocket / gRPC inspection

### MVP Success Criteria
- User có thể capture HTTPS traffic trong vòng **3 phút** kể từ lần cài đầu tiên
- Ad traffic bị block tự động mà không cần config thêm
- Filter/search hoạt động với >1000 requests mà không lag

## User Stories

| As a | I want to | So that | Priority |
|------|-----------|---------|----------|
| Solo developer | Xem HTTPS traffic của app tôi đang build | Tôi có thể debug API calls mà không cần setup phức tạp | P0 |
| Solo developer | Tự động block ad/tracker requests | Traffic log chỉ hiện request liên quan đến app của tôi | P0 |
| QA engineer | Filter requests theo domain hoặc status code | Tôi tìm được request cần test nhanh hơn | P0 |
| Developer | Cài app và bắt đầu capture trong < 3 phút | Tôi không mất thời gian vào setup | P0 |
| Developer | Xem chi tiết headers và body của từng request | Tôi debug được lỗi API chính xác hơn | P1 |
| QA engineer | Export traffic session ra file | Tôi share được bug report với developer | P1 |

## Business Model

### Pricing Model
- [x] Freemium

### Pricing Tiers

| Tier | Price | Features | Target |
|------|-------|----------|--------|
| Free | $0 | Proxy + inspector + ad blocking, unlimited requests | Solo dev, students |
| Pro | $49/năm | Request mocking, export sessions, rule engine, priority support | QA engineer, power user |
| Enterprise | Custom | Team license, SSO, audit log | Dev team 10+ người |

### Unit Economics (Estimated)
- CAC: ~$5 (organic — GitHub, HN, Reddit)
- LTV (Pro): ~$98 (2 năm average retention)
- LTV:CAC ratio: ~20:1
- Payback period: < 1 tháng
- Target: 1000 Pro users năm đầu = ~$49K ARR

## Moat / Defensibility

| Moat Type | Strength | Description |
|-----------|----------|-------------|
| Network effects | LOW | Tool cá nhân, không có network effect tự nhiên |
| Data | LOW | Local-only, không collect user data (feature, không phải bug) |
| Switching costs | MEDIUM | Khi user đã quen workflow + custom rules, ngại chuyển tool |
| Brand | MEDIUM | First-mover trong "free HTTP inspector + ad blocker" niche |
| Technology | LOW | Stack không độc quyền, competitor có thể copy |

> Moat chính là **brand + community** — build OSS, grow GitHub stars, trở thành go-to tool cho emerging market developers.

## Risks & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|-----------|
| mitmproxy ra GUI chính thức | MEDIUM | HIGH | Ship nhanh, build community trước khi họ kịp react |
| Browser DevTools ngày càng mạnh | HIGH | MEDIUM | Focus vào non-browser traffic (mobile, desktop apps) |
| SSL/TLS complexity tăng, harder to intercept | MEDIUM | HIGH | Invest vào certificate management UX, document rõ ràng |
| Monetization khó — user không upgrade | HIGH | MEDIUM | Validate Pro features với QA segment trước khi build |
| Legal/privacy concerns | LOW | HIGH | Rõ ràng: chỉ local traffic, không gửi data ra ngoài, open source |

## Go / No-go

- **Decision:** CONDITIONAL GO
- **Reasoning:** Gap thị trường có thật, stack để build đã sẵn sàng, CAC thấp nhờ organic distribution. Tuy nhiên moat mỏng — thành bại phụ thuộc vào tốc độ ship và chất lượng UX, không phải tính năng.
- **Conditions:**
  - Validate với ít nhất 5 developer thực tế rằng họ sẽ dùng tool này thay Charles/mitmproxy
  - Confirm Pro features ($49/năm) đủ hấp dẫn để QA engineer upgrade
  - Ship MVP trong 6-8 tuần, không scope creep

## Assumptions
- Developer ở emerging markets (SEA, India) là early adopter chính — chưa validate trực tiếp
- EasyList/uBlock filter lists đủ để block phần lớn ad traffic mà không cần maintain custom list
- Freemium conversion rate ~2-5% (industry average cho developer tools)
- Organic distribution qua GitHub/HN đủ để reach 10K users năm đầu

## Next Steps
- [ ] Phỏng vấn 5 developer về workflow debug hiện tại và pain points
- [ ] Build prototype: proxy + basic UI (2 tuần)
- [ ] Test one-click SSL setup trên Windows, macOS, Linux
- [ ] Post lên HN "Show HN" để validate interest
- [ ] Quyết định tech stack: Tauri (Rust) vs Electron (Node.js)
