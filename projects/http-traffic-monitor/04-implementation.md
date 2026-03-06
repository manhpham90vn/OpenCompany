# FlowProxy - Implementation Plan

> Date: 2026-03-06
> Input: `../03-technical-architecture.md`
> Status: draft

## Project Setup

- **Repo**: https://github.com/flowproxy/flowproxy (to be created)
- **Branch strategy**: Trunk-based development (main branch + feature branches)
- **Dev environment**:
  - Rust 1.75+ stable
  - Node.js 20+ LTS
  - pnpm 8+
  - Tauri CLI 2.x
  - Run: `pnpm install` → `tauri dev` (hot reload enabled)
- **Staging**: GitHub Actions build artifacts (manual download for QA)
- **Production**: GitHub Releases

## CI/CD Pipeline

| Stage | Tool | Trigger | Actions |
|-------|------|---------|---------|
| Lint | clippy + eslint | on push | Rust clippy deny warnings, ESLint + Prettier |
| Test | cargo test + vitest | on push | Unit tests for Rust + React components |
| Build | tauri build (matrix) | on merge to main | Cross-platform: Win (.msi), macOS (.dmg), Linux (.AppImage + .deb) |
| Release | GitHub Releases | on tag push `v0.x.x` | Upload artifacts + auto-updater config |

## Testing Strategy

| Type | Tool | Coverage Target | Status |
|------|------|----------------|--------|
| Unit | cargo test (Rust), vitest (React) | 80%+ on core logic | To be implemented |
| Integration | Rust integration tests | Proxy capture, SSL handshake, ad blocking | To be implemented |
| E2E | Playwright | Happy path: start proxy → capture → view request | To be implemented |

## Sprint Plan

### Sprint 1 - Proxy Core (Week 1-2)

| Task | Priority | Status | Assignee | Notes |
|------|----------|--------|----------|-------|
| Initialize Tauri 2.x project with React + TypeScript | P0 | pending | | Use `pnpm create tauri-app` |
| Set up logging: tracing crate + panic handler | P0 | pending | | Log to app data directory |
| Implement HTTP proxy with hyper + tokio | P0 | pending | | Bind localhost:8080 |
| Implement CONNECT tunnel for HTTPS passthrough | P0 | pending | | Handle CONNECT method |
| Implement system proxy config (Win/Mac/Linux) | P0 | pending | | Platform-specific APIs |
| Set up SQLite with sqlx + WAL mode | P0 | pending | | Request storage |
| Implement Tauri IPC commands: `start_proxy`, `stop_proxy`, `get_requests` | P0 | pending | | Frontend ↔ Backend communication |
| Set up CI: GitHub Actions workflow | P0 | pending | | lint + test + build matrix |

**Sprint 1 Goals:**
- [ ] Proxy captures HTTP traffic from system
- [ ] Traffic stored in SQLite
- [ ] Basic `tauri dev` hot reload working

### Sprint 2 - SSL + Ad Blocking (Week 3)

| Task | Priority | Status | Assignee | Notes |
|------|----------|--------|----------|-------|
| Implement CA certificate generation with rcgen | P0 | pending | | Generate RSA 2048-bit CA |
| Implement OS certificate install (Windows) | P0 | pending | | CertOpenSystemStore + CertAddEncodedCertificateToSystemStore |
| Implement OS certificate install (macOS) | P0 | pending | | Security framework + Keychain |
| Implement OS certificate install (Linux) | P0 | pending | | ~/.local/share/ca-certificates or update-ca-certificates |
| SSL setup UI wizard | P0 | pending | | Step-by-step guide in React |
| Implement adblock-rs integration | P0 | pending | | Load EasyList/uBlock filter lists |
| Implement `toggle_ad_blocking`, `update_filter_lists` commands | P0 | pending | | Control ad blocking |
| Test SSL setup on Win 10/11, macOS 13/14, Ubuntu 22/24 | P0 | pending | | Manual QA on all platforms |

**Sprint 2 Goals:**
- [ ] User can capture HTTPS traffic within 3 minutes
- [ ] Ad/tracker requests blocked automatically

### Sprint 3 - Traffic Inspector UI (Week 4-5)

| Task | Priority | Status | Assignee | Notes |
|------|----------|--------|----------|-------|
| Set up Zustand for state management | P0 | pending | | |
| Configure shadcn/ui + Tailwind CSS | P0 | pending | | |
| Implement virtual list component | P0 | pending | | Use react-virtual for 1000+ requests |
| Connect real-time Tauri events | P0 | pending | | `request_captured`, `request_blocked` |
| Implement filter bar: host, method, status code | P0 | pending | | Search within session |
| Implement session management UI | P1 | pending | | Start/stop session, session list |

**Sprint 3 Goals:**
- [ ] Traffic list shows requests in real-time
- [ ] Filter/search works smoothly with 1000+ requests

### Sprint 4 - Request Detail View (Week 5.5)

| Task | Priority | Status | Assignee | Notes |
|------|----------|--------|----------|-------|
| Implement request detail panel | P0 | pending | | Headers + body viewer |
| Add timing breakdown display | P0 | pending | | DNS, connect, TLS, wait, receive |
| Add size formatting | P0 | pending | | Bytes/KB/MB auto-format |
| Implement `get_request_detail` command | P0 | pending | | Fetch full request data |

**Sprint 4 Goals:**
- [ ] User can view full request/response details
- [ ] Timing and size information clear

### Sprint 5 - Packaging + Launch (Week 6-7)

| Task | Priority | Status | Assignee | Notes |
|------|----------|--------|----------|-------|
| Configure Tauri bundler for all platforms | P0 | pending | | .msi, .dmg, .AppImage, .deb |
| Set up code signing (Windows + macOS) | P0 | pending | | $99 Apple + ~$300 Windows cert |
| Configure Tauri auto-updater | P0 | pending | | GitHub Releases endpoint |
| Performance testing: proxy latency | P0 | pending | | Target < 10ms overhead |
| Performance testing: virtual list stress | P0 | pending | | 5000+ requests without lag |
| Security audit: CA key permissions | P0 | pending | | chmod 600 on private key |
| Security audit: system proxy cleanup | P0 | pending | | Restore on crash |
| Write runbook documentation | P1 | pending | | User guide + troubleshooting |
| Create GitHub Releases + landing page | P1 | pending | | README, screenshots, demo |

**Sprint 5 Goals:**
- [ ] Signed installers for Win/Mac/Linux
- [ ] Auto-update functional
- [ ] MVP ready for release

## Progress Tracking

- [ ] Project scaffolding (Sprint 1)
- [ ] HTTP/HTTPS proxy core (Sprint 1)
- [ ] SSL certificate setup (Sprint 2)
- [ ] Ad/tracker blocking (Sprint 2)
- [ ] Traffic inspector UI (Sprint 3)
- [ ] Request detail view (Sprint 4)
- [ ] Cross-platform packaging (Sprint 5)
- [ ] Security audit (Sprint 5)
- [ ] Performance testing (Sprint 5)
- [ ] GitHub Release (Sprint 5)

## Dependencies & Blockers

| Dependency | Status | Impact | Owner |
|-----------|--------|--------|-------|
| hudsucker crate maintenance | Unknown | HIGH - may need mitmproxy-rs or custom hyper implementation | Self |
| Code signing certificates | Not yet purchased | MEDIUM - required for production release | Self |
| Test devices (Win 10/11, macOS 13/14, Ubuntu 22/24) | Available | HIGH - need manual QA for SSL install | Self |

## Launch Checklist

- [ ] Performance tested (proxy latency < 10ms, virtual list handles 1000+)
- [ ] Security audit done (CA key permissions, system proxy cleanup)
- [ ] Monitoring configured (tracing → local log files)
- [ ] Backup/restore verified (SQLite export/import)
- [ ] Runbook documented (README + troubleshooting guide)
- [ ] Rollback plan ready (revert to previous GitHub release tag)
- [ ] SSL certificates (not applicable - local CA)
- [ ] Error tracking setup (panic handler → local log)
- [ ] Analytics setup (NOT - privacy-first is selling point)

## Known Issues / Tech Debt

| Issue | Severity | Plan | Sprint |
|-------|----------|------|--------|
| hudsucker crate may be unmaintained | HIGH | Evaluate in Sprint 1; switch to custom hyper or mitmproxy-rs if needed | Sprint 1 |
| System proxy restore on crash | HIGH | Implement cleanup hook in Rust (Drop trait) | Sprint 1 |
| SSL cert install differences across OS versions | HIGH | Document fallback manual steps; test early | Sprint 2 |
| Large response body storage | MEDIUM | Lazy load bodies; consider streaming | Sprint 4 |
| Code signing cost ($400/year) | MEDIUM | Budget for certificates | Sprint 5 |

## Deployment Runbook

### Building Release

1. Ensure version bumped in `Cargo.toml` and `package.json`
2. Create git tag: `git tag -a v0.1.0 -m "v0.1.0"`
3. Push tag: `git push origin v0.1.0`
4. GitHub Actions triggers: lint → test → build matrix
5. Download artifacts from Actions run
6. Create GitHub Release with artifacts + release notes

### Installing for Development

```bash
# Install dependencies
pnpm install

# Run in development mode (hot reload)
tauri dev
```

### Installing for Production

1. Download installer from GitHub Releases
2. Run installer (Windows .msi / macOS .dmg / Linux .AppImage)
3. Launch app
4. Follow SSL setup wizard

### Troubleshooting

- **Proxy not capturing traffic**: Check system proxy is set to localhost:8080
- **HTTPS not showing**: Run SSL setup wizard again, check cert installed
- **Too many requests**: Use filter bar to narrow down
- **App crash on exit**: Check logs in app data directory

## Rollback Plan

1. **Identify issue**: Check logs in app data directory (`%APPDATA%\FlowProxy\logs` / `~/Library/Application Support/FlowProxy/logs` / `~/.config/FlowProxy/logs`)
2. **Stop proxy**: Click stop button or force quit app
3. **Restore system proxy**: Manually reset OS proxy settings if not auto-restored
4. **Revert version**: Download previous release from GitHub Releases
5. **Report issue**: Open GitHub Issue with logs attached
