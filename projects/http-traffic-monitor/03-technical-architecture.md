# FlowProxy - Technical Architecture

> Date: 2026-03-06
> Input: `../02-product-strategy.md`
> Status: draft

## Tech Stack

| Layer | Choice | Rationale | Alternatives Considered |
|-------|--------|-----------|------------------------|
| Desktop framework | Tauri 2.x | ~10MB bundle vs Electron ~100MB; Rust backend ideal for proxy | Electron (Node.js) — easier but heavier and slower |
| Frontend | React 18 + TypeScript + Vite | Large ecosystem, fast dev loop, virtual list great for 1000+ requests | Vue 3 — comparable but less resources |
| UI components | shadcn/ui + Tailwind CSS | Fast build, modern, no complex licensing | Ant Design — heavier, not needed |
| State management | Zustand | Lightweight, no boilerplate, sufficient for desktop app | Redux — over-engineered for this scope |
| Proxy core | hyper + tokio + rustls | Industry-standard async HTTP in Rust | reqwest — wrapper, less control |
| MITM / SSL interception | hudsucker | Dedicated MITM proxy library for Rust | mitmproxy-rs — less documented |
| Certificate generation | rcgen | Pure Rust, generate CA cert without OpenSSL | openssl crate — heavy dependency |
| Ad blocking engine | adblock-rs (Brave) | Parse EasyList/uBlock format natively | Self-implement — unnecessary time sink |
| Local storage | SQLite via sqlx | Embedded, zero-config, fast for traffic log | sled — less mature for relational data |
| Packaging | Tauri bundler + GitHub Actions | Automatic cross-platform builds (.msi, .dmg, .AppImage) | — |
| Auto-update | Tauri updater | Built-in, no separate server needed | Squirrel — Electron only |

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

1. App starts → Rust proxy server binds to `localhost:8080`
2. App sets OS system proxy → `localhost:8080` (platform-specific API)
3. Traffic from all apps → Proxy → adblock-rs check → forward or block
4. Each request saved to SQLite → emit Tauri event → frontend updates in real-time
5. App closes → system proxy restored to original settings

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

-- Indexes for fast filter/search
CREATE INDEX idx_requests_session ON requests(session_id);
CREATE INDEX idx_requests_host ON requests(host);
CREATE INDEX idx_requests_timestamp ON requests(timestamp);
CREATE INDEX idx_requests_is_blocked ON requests(is_blocked);
```

> SQLite WAL mode enabled by default to ensure writes don't block reads during high traffic.

## API Design (Tauri IPC Commands)

No HTTP API — communicate via Tauri commands and events.

### Tauri Commands (Frontend → Backend)

| Command | Input | Output | Description |
|---------|-------|--------|-------------|
| `start_proxy` | `{ port: u16 }` | `Result<()>` | Start proxy server |
| `stop_proxy` | — | `Result<()>` | Stop proxy, restore system proxy |
| `get_requests` | `{ session_id, filter?, limit?, offset? }` | `Vec<Request>` | Get request list |
| `get_request_detail` | `{ id }` | `RequestDetail` | Get full headers + body |
| `clear_session` | `{ session_id }` | `Result<()>` | Clear traffic log |
| `export_session` | `{ session_id, path }` | `Result<()>` | Export to HAR/JSON file |
| `install_certificate` | — | `Result<()>` | Generate + install CA cert |
| `get_filter_rules` | — | `Vec<FilterRule>` | Get filter rules list |
| `toggle_ad_blocking` | `{ enabled: bool }` | `Result<()>` | Enable/disable ad blocking |
| `update_filter_lists` | — | `Result<()>` | Reload EasyList from file |

### Tauri Events (Backend → Frontend)

| Event | Payload | Description |
|-------|---------|-------------|
| `request_captured` | `RequestSummary` | Emitted on each new request |
| `request_blocked` | `{ id, url, reason }` | Emitted when request is blocked |
| `proxy_status` | `{ running: bool, port: u16 }` | Proxy server status |
| `cert_install_status` | `{ success: bool, message }` | Certificate install result |

## Third-party Integrations

| Service | Purpose | Cost | Setup Effort |
|---------|--------|------|--------------|
| EasyList (CDN) | Filter list for ad blocking | Free | Low — download file, parse offline |
| uBlock Origin filter lists | Additional tracker blocking | Free | Low — same Adblock Plus format |
| GitHub Releases | Binary distribution | Free | Low — GitHub Actions integration |
| Tauri updater | Auto-update | Free | Medium — need to sign binary |

> No cloud services needed for MVP. All processing is local.

## MVP Technical Scope

### Features → Technical Tasks

| Feature | Technical Tasks | Effort |
|---------|----------------|--------|
| Local HTTP/HTTPS proxy | Hyper server, system proxy config (Win/Mac/Linux), CONNECT tunnel | 2 weeks |
| One-click SSL setup | rcgen CA generation, OS cert install (3 platforms), UI wizard | 1 week |
| Ad/tracker blocking | adblock-rs integration, EasyList download + parse, toggle UI | 1 week |
| Traffic inspector UI | Virtual list component, real-time Tauri events, filter/search bar | 1.5 weeks |
| Request detail view | Headers/body viewer, timing display, size formatting | 0.5 week |
| Cross-platform packaging | GitHub Actions matrix build, code signing, auto-updater | 1 week |

**Total: ~7 weeks**

### Out of Scope (MVP)
- Request modification / mocking
- Export session (P1 — after MVP)
- Team collaboration / cloud sync
- WebSocket / gRPC inspection
- Plugin ecosystem
- Mobile companion app

### MVP Technical Requirements
- Capture HTTPS traffic in < 3 minutes from first install
- Filter/search works smoothly with > 1000 requests (virtual list)
- Proxy latency overhead < 10ms for normal requests
- Bundle size < 20MB (installer)

## Security

- **CA Certificate**: Only install locally, never send externally. Private key stored in app data directory with permission 600.
- **Traffic data**: All local, no telemetry, no cloud sync.
- **Open source**: Transparency is the main moat — users can audit code.
- **System proxy**: Restore to original settings when app crashes (cleanup hook).
- **Body storage**: Request/response bodies may contain sensitive data — consider encrypting SQLite file (SQLCipher) at Pro tier.
- **Compliance**: GDPR doesn't apply (no data collection). Need clear disclaimer: "only use to debug your own traffic".

## Infrastructure

### Environments

| Env | Purpose | Config |
|-----|---------|--------|
| dev | Local development | `tauri dev`, hot reload frontend |
| staging | Pre-release testing | GitHub Actions build, manual QA |
| prod | GitHub Releases | Signed binary, auto-updater endpoint |

> No server infrastructure needed. App runs entirely locally.

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

- **Crash reporting**: Tauri panic handler → write log file locally (no external sending)
- **App logs**: `tracing` crate → log file in app data directory
- **GitHub Issues**: User-reported bugs (no automated crash reporting for MVP)
- **Analytics**: None — privacy-first is a selling point

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
| SSL cert install differs across OS versions | HIGH | HIGH | Test early on Win 10/11, macOS 13/14, Ubuntu 22/24. Document fallback manual steps |
| System proxy API quirks on Linux | MEDIUM | HIGH | Support gsettings (GNOME) + KDE + env vars + PAC file |
| hudsucker crate is poorly maintained | MEDIUM | MEDIUM | Evaluate mitmproxy-rs or build directly on hyper if needed |
| Performance with high traffic (>500 req/s) | MEDIUM | MEDIUM | SQLite WAL + in-memory ring buffer for active session, lazy load body |
| Code signing cost (Windows/macOS) | LOW | MEDIUM | $99/year Apple Developer + ~$300/year Windows cert — factor into budget |
| Users concerned about MITM proxy | LOW | HIGH | Open source + clear documentation + local-only messaging |

## Effort Estimate

| Milestone | Tasks | Estimate | Dependencies |
|-----------|-------|----------|--------------|
| M1: Proxy core | HTTP proxy + CONNECT tunnel + system proxy config | 2 weeks | — |
| M2: SSL setup | CA cert gen + OS install (3 platforms) | 1 week | M1 |
| M3: Ad blocking | adblock-rs + EasyList integration | 1 week | M1 |
| M4: UI - Traffic list | Virtual list + real-time events + filter/search | 1.5 weeks | M1 |
| M5: UI - Detail view | Headers/body viewer + timing | 0.5 week | M4 |
| M6: Packaging | GitHub Actions + signing + auto-updater | 1 week | M1-M5 |

**Total MVP: ~7 weeks (1 developer)**

## Scaling Plan

| Stage | Trigger | Changes |
|-------|---------|---------|
| MVP (local) | 0 → 10K users | No server needed. Just GitHub Releases bandwidth |
| Pro tier | 1K Pro users | Add simple license server (Cloudflare Worker + KV) to validate key |
| Team features | 10K+ users | Optional cloud sync — S3 + simple API. Still local-first |
| Enterprise | 100+ team seats | Self-hosted option, SSO integration |

## Development Guidelines

### Code Standards
- Rust: stable channel, `clippy` with `#![deny(warnings)]`
- TypeScript: strict mode, ESLint + Prettier
- Testing: `cargo test` for Rust logic, `vitest` for frontend components
- Commit: Conventional Commits (`feat:`, `fix:`, `chore:`)

### Git Workflow
- Branch: `main` (production) + feature branches
- PR: require 1 review + CI pass
- Release: tag `v0.x.x` → GitHub Actions auto build + publish
