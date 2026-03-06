# [Tên dự án] - Technical Architecture

> Ngày: YYYY-MM-DD
> Input: `../02-product-strategy/[file].md`
> Status: draft | reviewed | approved

## Tech Stack

| Layer | Choice | Lý do | Alternatives Considered |
|-------|--------|-------|----------------------|
| Frontend | | | |
| Backend | | | |
| Database | | | |
| Auth | | | |
| Hosting | | | |
| CI/CD | | | |
| Monitoring | | | |
| Payments | | | |

## System Architecture

<!-- Mô tả kiến trúc tổng quan, có thể dùng ASCII diagram -->

```
[Client] --> [API Gateway] --> [Services] --> [Database]
```

### Architecture Diagram
<!-- Vẽ diagram chi tiết hơn nếu cần -->

## Data Models

### Core Entities

| Entity | Fields | Relationships |
|--------|--------|---------------|
| | | |

### Database Schema
<!-- Chi tiết hơn về tables, indexes -->

## API Design

### Core Endpoints

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | /api/... | | |
| POST | /api/... | | |

### API Contracts
<!-- OpenAPI spec link or summary -->

## Third-party Integrations

| Service | Mục đích | Cost estimate | Setup effort |
|---------|---------|---------------|--------------|
| | | $ | |

## MVP Technical Scope

### Features → Technical Tasks

| Feature | Technical Tasks | Effort |
|---------|----------------|--------|
| | | |

### Out of Scope
- [Feature]
- [Feature]

### MVP Technical Requirements
- Min users:
- Max latency:
- Uptime target:

## Security

- Auth strategy: [JWT / Session / OAuth]
- Data protection: [Encryption at rest/transit]
- Compliance: [GDPR, SOC2, etc.]
- Rate limiting:
- API security:

## Infrastructure

### Environments
| Env | Purpose | Config |
|-----|---------|--------|
| dev | Local development | |
| staging | Pre-production testing | |
| prod | Production | |

### Monitoring & Logging
- Error tracking:
- Performance monitoring:
- Logging:

### Backup & Recovery
- Backup strategy:
- RTO/RPO:
- Disaster recovery plan:

## Risks & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|-----------|
| | | | |

## Effort Estimate

| Milestone | Tasks | Estimate (hours) | Dependencies |
|-----------|-------|------------------|--------------|
| | | | |

## Scaling Plan

| Scale Stage | Trigger | Architecture Changes |
|-------------|---------|---------------------|
| 100 users | | |
| 1,000 users | | |
| 10,000 users | | |

## Development Guidelines

### Code Standards
- Language version:
- Linting:
- Testing framework:

### Git Workflow
- Branch strategy:
- PR requirements:
- Deployment process:
