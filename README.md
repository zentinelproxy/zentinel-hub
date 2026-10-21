<p align="center">
  <a href="https://zentinelproxy.io">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/zentinelproxy/design/main/marks/lockup/zentinel-lockup-dark.svg">
      <img alt="Zentinel" src="https://raw.githubusercontent.com/zentinelproxy/design/main/marks/lockup/zentinel-lockup-light.svg" height="52">
    </picture>
  </a>
</p>

<h3 align="center">Zentinel Hub (deprecated)</h3>

<p align="center">An earlier, Go-based fleet manager for Zentinel proxies, superseded by the Zentinel control plane.</p>

<p align="center">
  <img alt="Go" src="https://img.shields.io/badge/go-1.25%2B-f68d31?style=flat&labelColor=1f161e&logo=go&logoColor=white">
</p>

<p align="center">
  <a href="https://github.com/zentinelproxy/zentinel-control-plane">Zentinel control plane</a> &middot;
  <a href="https://docs.zentinelproxy.io">Docs</a> &middot;
  <a href="https://github.com/zentinelproxy">All repositories</a>
</p>

> **Deprecated.** Zentinel Hub is no longer developed. Fleet management lives
> in [zentinel-control-plane](https://github.com/zentinelproxy/zentinel-control-plane).
> This repository is kept for reference; do not deploy it.

Zentinel Hub was a pre-alpha control plane: a Go API server (`hub`) with a
Next.js web UI, and a small sidecar (`agent`, not a Zentinel agent in the
agent-protocol sense) that runs next to each proxy, connects to the hub over
gRPC, registers, sends heartbeats and receives configuration and deployment events. The hub versions configurations, tracks instances and
records deployments and an audit log.

## How it works

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Web UI    │────▶│  Hub API    │────▶│  Database   │
│  (Next.js)  │     │    (Go)     │     │ (SQLite/PG) │
└─────────────┘     └──────┬──────┘     └─────────────┘
                           │ gRPC
         ┌─────────────────┼─────────────────┐
         ▼                 ▼                 ▼
    ┌─────────┐       ┌─────────┐       ┌─────────┐
    │ sidecar │       │ sidecar │       │ sidecar │
    └────┬────┘       └────┬────┘       └────┬────┘
    ┌────┴────┐       ┌────┴────┐       ┌────┴────┐
    │Zentinel │       │Zentinel │       │Zentinel │
    └─────────┘       └─────────┘       └─────────┘
```

## Configuration

`hub serve` flags:

| Option | Default | Meaning |
|---|---|---|
| `--http-port` | `8080` | HTTP API port |
| `--grpc-port` | `9090` | gRPC port for sidecars |
| `--database-url` | `sqlite://hub.db` | Database URL |

Environment: `HUB_JWT_SECRET` (falls back to an insecure development secret
with a warning), `HUB_ADMIN_EMAIL` / `HUB_ADMIN_PASSWORD` / `HUB_ADMIN_NAME`
(seed an admin user), `HUB_LOG_FORMAT=json`.

`agent run` flags: `--hub-url` (`localhost:9090`), `--instance-id`,
`--instance-name` (hostname), `--zentinel-config`
(`/etc/zentinel/config.kdl`), `--zentinel-version`, `--heartbeat-interval`
(`30` seconds), `--label key=value` (repeatable). Environment:
`AGENT_LOG_LEVEL`, `AGENT_LOG_FORMAT`.

## API

```
GET  /health, /ready, /stats
POST /api/v1/auth/login | /auth/refresh | /auth/logout     GET /api/v1/auth/me
GET  /api/v1/instances[/{id}]       POST /api/v1/instances      PUT|DELETE /api/v1/instances/{id}
GET  /api/v1/configs[/{id}]         POST /api/v1/configs        PUT|DELETE /api/v1/configs/{id}
GET  /api/v1/configs/{id}/versions  POST /api/v1/configs/{id}/rollback
GET  /api/v1/deployments[/{id}]     POST /api/v1/deployments    POST /api/v1/deployments/{id}/cancel
     /api/v1/users[/{id}]           POST /api/v1/users/{id}/reset-password
GET  /api/v1/audit-logs
```

## Development

Requires [mise](https://mise.jdx.dev/) (Go 1.25.5 and Node 22 are pinned in
`mise.toml`).

```bash
mise install
mise run setup        # Go modules and web dependencies
mise run dev          # go run ./cmd/hub serve
mise run dev:web      # Next.js dev server
mise run dev:all      # both
mise run build        # bin/hub and bin/agent
mise run test         # go tests
mise run check        # fmt, lint, test
```

There is no CI workflow in this repository.

## Related

- [zentinel-control-plane](https://github.com/zentinelproxy/zentinel-control-plane): the replacement
- [Zentinel](https://github.com/zentinelproxy/zentinel): the proxy
- [docs/fleet-management.md](docs/fleet-management.md), [.claude/ARCHITECTURE.md](.claude/ARCHITECTURE.md), [.claude/ROADMAP.md](.claude/ROADMAP.md): design notes from when it was active

## License

This repository has no `LICENSE` file.

---

<p align="center">
  <sub>Part of <a href="https://zentinelproxy.io">Zentinel</a>, the security-first reverse proxy. The shisa guards the gate.</sub>
</p>
