# FlowProxy - Technical Architecture

> Date: 2026-03-06
> Input: `../02-product-strategy.md`
> Status: draft

## Tech Stack

| Layer | Choice | Rationale | Alternatives Considered |
|-------|--------|-----------|------------------------|
| Desktop framework | Tauri 2.x | Bundle ~10MB vs Electron ~100MB; Rust backend lý tưởng cho proxy | Electron (Node.js) — dễ hơn nhưng nặng và chậm hơn |
| Frontend | React 18 + TypeScript + Vite | Ecosystem lớn, dev loop nhanh, virtual list tốt cho 1000+ requests | Vue 3 — tương đương nhưng ít tài nguyên hơn |
| UI components | shadcn/ui + Tailwind CSS | Build nhanh, modern, không license phức tạp | Ant Design — nặng hơn, không cần thiết |
| State management | Zustand | Nhẹ, không boilerplate, đủ cho desktop app | Redux — over-engineer cho scope này |
| Proxy core | hyper + tokio + rustls | Async HTTP chuẩn công nghiệp trong Rust | reqwest — wrapper, ít control hơn |
| MITM / SSL interception | hudsucker | Thư viện MITM proxy chuyên dụng cho Rust | mitmproxy-rs — ít tài liệu hơn |
| Certificate generation | rcgen | Pure Rust, tạo CA cert không cần OpenSSL | openssl crate — dependency nặng |
| Ad blocking engine | adblock-rs (Brave) | Parse EasyList/uBlock format native | Tự implement — tốn thời gian không cần thiết |
| Local storage | SQLite via sqlx | Embedded, zero-config, nhanh cho traffic log | sled — ít mature hơn cho relational data |
| Packaging | Tauri bundler + GitHub Actions | Cross-platform build tự động (.msi, .dmg, .AppImage) | — |
| Auto-update | Tauri updater | Built-in, không cần server riêng | Squirrel — chỉ cho Electron |

## System Architecture

```
┌──────────────────────────────────────────────────────────┐
│                   FlowProxy (Tauri 2.x App)              │
│                                                          │
│  ┌──────────────────────┐  IPC   ┌────────────────────┐ │
│  │   React Frontend     │◄──────►│   Rust Backend     │ │
│  │   (WebView)          │ events │                    │ │
│  │                      │        │  ┌──────────────┐  │ │
│  │  - Traffic list      │        │  │ Proxy Server │  │ │
│  │  - Filter/search bar │        │  │ hyper+tokio  │  │ │
│  │  - Request detail    │        │  └──────┬───────┘  │ │
│  │  - Settings panel    │        │         │          │ │
│  └──────────────────────┘        │  ┌──────▼───────┐  │ │
│                                  │  │  Ad Blocker  │  │ │
│                                  │  │  adblock-rs  │  │ │
│                                  │  └──────┬───────┘  │ │
│                                  │         │          │ │
│                                  │  ┌──────▼───────┐  │ │
│                                  │  │ SQLite Store │  │ │
│                                  │  │ sqlx + WAL   │  │ │
│                                  │  └──────────────┘  │ │
│                                  │  ┌──────────────┐  │ │
│                                  │  │ Cert Manager │  │ │
│                                  │  │ rcgen        │  │ │
│                                  │  └──────────────┘  │ │
│                                  └────────────────────┘ │
└──────────────────────────────────────────────────────────┘
          │ set system proxy
          ▼
   localhost:8080
          │
          ▼
      Internet
```

### Core Data Flow

1. App khởi động → Rust proxy server bind `localhost:8080`
2. App set OS system proxy → `localhost:8080` (platform-specific API)
3. Traffic từ mọi app → Proxy → adblock-rs check → forward hoặc block
4. Mỗi request lưu vào SQLite → emit Tauri event → frontend cập nhật real-time
5. App đóng → system proxy restore về cài đặt gốc

## Data Models

### Core Entities

| Entity | Fields | Relationships |
|--------|--------|---------------|
| Request | id, timestamp, method, url, host, path, status_code, request_headers, response_headers, request_body, response_body, duration_ms, size_bytes, is_blocked, blocked_reason | belongs to Session |
| Session | id, name, created_at, ended_at, request_count | has many Requests |
| FilterRule | id, pattern, rule_type (block/allow), source (easylists/custom), enabled, created_at | — |

### Database Schema

```sql
CREATE TABLE sessions (
  id          TEXT PRIMARY KEY,
  name        TEXT NOT NULL,
  created_at  INTEGER NOT NULL,
  ended_at    INTEGER,
  request_count INTEGER DEFAULT 0
);

CREATE TABLE requests (
  id               TEXT PRIMARY KEY,
  session_id       TEXT NOT NULL REFERENCES sessions(id),
  timestamp        INTEGER NOT NULL,
  method           TEXT NOT NULL,
  url              TEXT NOT NULL,
  host             TEXT NOT NULL,
  path             TEXT NOT NULL,
  status_code      INTEGER,
  request_headers  TEXT,  -- JSON
  response_headers TEXT,  -- JSON
  request_body     BLOB,
  response_body    BLOB,
  duration_ms      INTEGER,
  size_bytes       INTEGER,
  is_blocked       INTEGER DEFAULT 0,
  blocked_reason   TEXT
);

CREATE TABLE filter_rules (
  id         TEXT PRIMARY KEY,
  pattern    TEXT NOT NULL,
  rule_type  TEXT NOT NULL,  -- 'block' | 'allow'
  source     TEXT NOT NULL,  -- 'easylists' | 'custom'
  enabled    INTEGER DEFAULT 1,
  created_at INTEGER NOT NULL
);

-- Indexes cho filter/search nhanh
CREATE INDEX idx_requests_session ON requests(session_id);
CREATE INDEX idx_requests_host ON requests(host);
CREATE INDEX idx_requests_timestamp ON requests(timestamp);
CREATE INDEX idx_requests_is_blocked ON requests(is_blocked);
```

> SQLite WAL mode bật mặc định để đảm bảo write không block read khi traffic cao.

## API Design (Tauri IPC Commands)

Không có HTTP API — giao tiếp qua Tauri commands và events.

### Tauri Commands (Frontend → Backend)

| Command | Input | Output | Description |
|---------|-------|--------|-------------|
| `start_proxy` | `{ port: u16 }` | `Result<()>` | Khởi động proxy server |
| `stop_proxy` | — | `Result<()>` | Dừng proxy, restore system proxy |
| `get_requests` | `{ session_id, filter?, limit?, offset? }` | `Vec<Request>` | Lấy danh sách requests |
| `get_request_detail` | `{ id }` | `RequestDetail` | Lấy headers + body đầy đủ |
| `clear_session` | `{ session_id }` | `Result<()>` | Xóa traffic log |
| `export_session` | `{ session_id, path }` | `Result<()>` | Export ra file HAR/JSON |
| `install_certificate` | — | `Result<()>` | Generate + install CA cert |
| `get_filter_rules` | — | `Vec<FilterRule>` | Lấy danh sách filter rules |
| `toggle_ad_blocking` | `{ enabled: bool }` | `Result<()>` | Bật/tắt ad blocking |
| `update_filter_lists` | — | `Result<()>` | Tải lại EasyList từ file |

### Tauri Events (Backend → Frontend)

| Event | Payload | Description |
|-------|---------|-------------|
| `request_captured` | `RequestSummary` | Emit mỗi khi có request mới |
| `request_blocked` | `{ id, url, reason }` | Emit khi request bị block |
| `proxy_status` | `{ running: bool, port: u16 }` | Trạng thái proxy server |
| `cert_install_status` | `{ success: bool, message }` | Kết quả cài certificate |

## Third-party Integrations

| Service | Purpose | Cost | Setup Effort |
|---------|---------|------|--------------|
| EasyList (CDN) | Filter list cho ad blocking | Free | Thấp — download file, parse offline |
| uBlock Origin filter lists | Bổ sung tracker blocking | Free | Thấp — cùng format Adblock Plus |
| GitHub Releases | Phân phối binary | Free | Thấp — tích hợp GitHub Actions |
| Tauri updater | Auto-update | Free | Trung bình — cần sign binary |

> Không có cloud service nào cần thiết cho MVP. Tất cả xử lý local.

## MVP Technical Scope

### Features → Technical Tasks

| Feature | Technical Tasks | Effort |
|---------|----------------|--------|
| Local HTTP/HTTPS proxy | Hyper server, system proxy config (Win/Mac/Linux), CONNECT tunnel | 2 tuần |
| One-click SSL setup | rcgen CA generation, OS cert install (3 platform), UI wizard | 1 tuần |
| Ad/tracker blocking | adblock-rs integration, EasyList download + parse, toggle UI | 1 tuần |
| Traffic inspector UI | Virtual list component, real-time Tauri events, filter/search bar | 1.5 tuần |
| Request detail view | Headers/body viewer, timing display, size formatting | 0.5 tuần |
| Cross-platform packaging | GitHub Actions matrix build, code signing, auto-updater | 1 tuần |

**Tổng: ~7 tuần**

### Out of Scope (MVP)
- Request modification / mocking
- Export session (P1 — sau MVP)
- Team collaboration / cloud sync
- WebSocket / gRPC inspection
- Plugin ecosystem
- Mobile companion app

### MVP Technical Requirements
- Capture HTTPS traffic trong < 3 phút từ lần cài đầu
- Filter/search hoạt động mượt với > 1000 requests (virtual list)
- Proxy latency overhead < 10ms cho request thông thường
- Bundle size < 20MB (installer)

## Security

- **CA Certificate**: Chỉ install local, không gửi ra ngoài. Private key lưu trong app data directory với permission 600.
- **Traffic data**: Tất cả local, không telemetry, không cloud sync.
- **Open source**: Transparency là moat chính — user có thể audit code.
- **System proxy**: Restore về cài đặt gốc khi app crash (cleanup hook).
- **Body storage**: Request/response body có thể chứa sensitive data — cân nhắc encrypt SQLite file (SQLCipher) ở Pro tier.
- **Compliance**: GDPR không áp dụng (không collect data). Cần disclaimer rõ ràng: "chỉ dùng để debug traffic của chính bạn".

## Infrastructure

### Environments

| Env | Purpose | Config |
|-----|---------|--------|
| dev | Local development | `tauri dev`, hot reload frontend |
| staging | Pre-release testing | GitHub Actions build, manual QA |
| prod | GitHub Releases | Signed binary, auto-updater endpoint |

> Không có server infrastructure. App chạy hoàn toàn local.

### Distribution

```
GitHub Actions (matrix build)
  ├── Windows  → .msi (code signed)
  ├── macOS    → .dmg (notarized)
  └── Linux    → .AppImage + .deb
          │
          ▼
    GitHub Releases
          │
          ▼
    Tauri Updater (check latest release tag)
```

### Monitoring & Logging

- **Crash reporting**: Tauri panic handler → write log file local (không gửi ra ngoài)
- **App logs**: `tracing` crate → file log trong app data directory
- **GitHub Issues**: User-reported bugs (không có automated crash reporting cho MVP)
- **Analytics**: Không có — privacy-first là selling point

### CI/CD Pipeline

```yaml
# GitHub Actions workflow
on: [push to main, PR]

jobs:
  test:     cargo test + vitest
  lint:     clippy + eslint
  build:    tauri build (matrix: ubuntu, windows, macos)
  release:  upload to GitHub Releases (on tag push)
```

## Risks & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|-----------|
| SSL cert install khác nhau giữa OS versions | HIGH | HIGH | Test sớm trên Win 10/11, macOS 13/14, Ubuntu 22/24. Document fallback manual steps |
| System proxy API quirks trên Linux | MEDIUM | HIGH | Hỗ trợ gsettings (GNOME) + KDE + env vars + PAC file |
| hudsucker crate ít maintained | MEDIUM | MEDIUM | Evaluate mitmproxy-rs hoặc build trực tiếp trên hyper nếu cần |
| Performance với traffic cao (>500 req/s) | MEDIUM | MEDIUM | SQLite WAL + in-memory ring buffer cho active session, lazy load body |
| Code signing cost (Windows/macOS) | LOW | MEDIUM | $99/năm Apple Developer + ~$300/năm Windows cert — tính vào budget |
| User lo ngại MITM proxy | LOW | HIGH | Open source + clear documentation + local-only messaging |

## Effort Estimate

| Milestone | Tasks | Estimate | Dependencies |
|-----------|-------|----------|--------------|
| M1: Proxy core | HTTP proxy + CONNECT tunnel + system proxy config | 2 tuần | — |
| M2: SSL setup | CA cert gen + OS install (3 platforms) | 1 tuần | M1 |
| M3: Ad blocking | adblock-rs + EasyList integration | 1 tuần | M1 |
| M4: UI - Traffic list | Virtual list + real-time events + filter/search | 1.5 tuần | M1 |
| M5: UI - Detail view | Headers/body viewer + timing | 0.5 tuần | M4 |
| M6: Packaging | GitHub Actions + signing + auto-updater | 1 tuần | M1-M5 |

**Tổng MVP: ~7 tuần (1 developer)**

## Scaling Plan

| Stage | Trigger | Thay đổi |
|-------|---------|---------|
| MVP (local) | 0 → 10K users | Không cần server. Chỉ cần GitHub Releases bandwidth |
| Pro tier | 1K Pro users | Thêm license server đơn giản (Cloudflare Worker + KV) để validate key |
| Team features | 10K+ users | Cloud sync optional — S3 + simple API. Vẫn local-first |
| Enterprise | 100+ team seats | Self-hosted option, SSO integration |

## Development Guidelines

### Code Standards
- Rust: stable channel, `clippy` với `#![deny(warnings)]`
- TypeScript: strict mode, ESLint + Prettier
- Testing: `cargo test` cho Rust logic, `vitest` cho frontend components
- Commit: Conventional Commits (`feat:`, `fix:`, `chore:`)

### Git Workflow
- Branch: `main` (production) + feature branches
- PR: require 1 review + CI pass
- Release: tag `v0.x.x` → GitHub Actions tự động build + publish
