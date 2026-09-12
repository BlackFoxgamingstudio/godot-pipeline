# Architecture: Sovereign Godot Pipeline

## Overview

**Package ID:** `PKG-008`  
**Domain:** Interactive Simulation & CI/CD  
**Microservice Port:** `8786`  
**n8n Webhook Path:** `godot-pipeline-trigger`  
**GitHub:** [BlackFoxgamingstudio/godot-pipeline](https://github.com/BlackFoxgamingstudio/godot-pipeline)

Godot Engine build automation pipeline with headless export, GUT test runner, asset pipeline, and n8n-triggered CI/CD release management.

---

## System Architecture

```
                     ┌──────────────────────────────────┐
                     │       Sovereign Godot Pipeline      │
                     │       Port: 8786            │
                     ├──────────────┬───────────────────┤
   n8n Webhook ────▶ │  REST API    │   Core Engine     │
   HTTP POST         │  /api/v1/*   │   Dispatcher      │
                     └──────┬───────┴────────┬──────────┘
                            │                │
              ┌─────────────▼────────────────▼─────────┐
              │          Component Layer                 │
              │  GodotExporter   | GUTTestRunner   | AssetPipelin  │
              └────────────────────────┬────────────────┘
                                       │
              ┌────────────────────────▼────────────────┐
              │      n8n Central Event Bus (:5678)       │
              └─────────────────────────────────────────┘
```

## Core Components

### `GodotExporter`
Handles all godotexporter operations. Exposes async methods callable from the core dispatcher.

### `GUTTestRunner`
Handles all guttestrunner operations. Exposes async methods callable from the core dispatcher.

### `AssetPipeline`
Handles all assetpipeline operations. Exposes async methods callable from the core dispatcher.

### `ReleaseBundler`
Handles all releasebundler operations. Exposes async methods callable from the core dispatcher.

### `CICDTrigger`
Handles all cicdtrigger operations. Exposes async methods callable from the core dispatcher.

---

## API Contract

All interactions follow the SBB standard envelope:

```http
POST /api/v1/execute
Content-Type: application/json
X-SBB-API-Key: <api-key>

{
  "action": "<operation>",
  "payload": {},
  "trace_id": "optional-uuid"
}
```

**Success Response (HTTP 200):**
```json
{
  "status": "success",
  "data": {},
  "trace_id": "...",
  "timestamp": "2025-01-01T00:00:00Z"
}
```

**Health Check:**
```http
GET /health
→ {"status": "healthy", "service": "sovereign-godot-pipeline", "port": 8786}
```

## Integration Matrix

| System | Protocol | Direction | Purpose |
|--------|----------|-----------|---------|
| n8n Event Bus (:5678) | HTTP POST | Outbound | Event forwarding |
| n8n Webhook | HTTP POST | Inbound | Trigger execution |
| SBB Codebase Vault (:8766) | HTTP | Outbound | Code analysis |
| SBB Patterns Bible (:8794) | HTTP | Outbound | Standards validation |
| External APIs | HTTPS | Outbound | Domain-specific data |

## Deployment Architecture

```yaml
# docker-compose excerpt
sovereign-godot-pipeline:
  image: sovereign-godot-pipeline:latest
  ports: ["8786:8786"]
  healthcheck:
    test: curl -f http://localhost:8786/health
    interval: 30s
```

## Security Model

| Control | Implementation |
|---------|---------------|
| Authentication | `X-SBB-API-Key` header (env: `SBB_API_KEY`) |
| Rate Limiting | 100 req/min per client IP |
| Input Validation | Pydantic models (strict mode) |
| Container Security | Non-root user (`appuser:1001`) |
| Secrets | Environment variables only (never hardcoded) |
| TLS | Terminate at reverse proxy (nginx/caddy) |

## Tags
`godot`, `game-dev`, `ci-cd`, `simulation`
