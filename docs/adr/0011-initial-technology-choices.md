# 0011. Initial technology choices

Date: 2026-09-22. Status: accepted.

## Context

Defaults chosen at kick-off so the first issues do not each re-open them.
Each can be revisited with a superseding record.

## Decision

| Area | Choice | Why |
|---|---|---|
| Go | 1.27; the standard `net/http` router | Pattern routing since 1.22 is enough; no framework |
| Database | PostgreSQL 17, `pgx`, `sqlc`, migrations with `goose` | Typed SQL is the idiomatic way to learn Go with a database |
| API | OpenAPI is the source of truth. Go server stubs from `oapi-codegen`; the TypeScript client from a separate generator (candidate: `openapi-typescript` + `openapi-fetch`), chosen in the API issue | The contract lets tools write the front end |
| Realtime | WebSocket via `coder/websocket`; in-process pub/sub on one server, replaced by Valkey pub/sub later behind an interface | Learn the single-server case first; keep the seam |
| Cache / pub-sub (later) | Valkey, not Redis | Project preference for the open-source fork |
| Infrastructure as code (later) | OpenTofu, not Terraform | Same |
| Logging | `log/slog` | No dependency |
| Tests | `testing`; `testcontainers-go` for a real PostgreSQL locally, service container in CI; k6 for load tests | |
| Lint | `golangci-lint`, `govulncheck` | |
| Front end | Svelte 5 + TypeScript, SvelteKit in static SPA mode; Tailwind with shared theme tokens by default, arbitrary values with justification; shadcn-svelte / Bits UI for complex widgets; `svelte-check`; Vitest; a few Playwright flows | One tool per layer, see ADR 0002 |
| Distribution | Multi-stage Docker build; `compose.yaml` with app + PostgreSQL; images on GHCR per tag | "Download and run" |
| Local layout | `repo/` is the main clone; `wt/` holds one worktree per issue | |

## Consequences

- The first issues are the repository skeleton around these choices, then CI,
  then database plumbing and migrations.
