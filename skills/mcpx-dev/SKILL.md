---
name: mcpx-dev
description: Build, test, lint, document and release changes to mcpx, the Go MCP security gateway (github.com/rohitgollarahalli/mcpx). Use for any work inside the mcpx repository — features, fixes, `mcpx scan` rules, `mcpx.yaml` config fields, transports, release prep — or when the user mentions mcpx or `mcpx scan` by name. Do not use for other gateways or proxies.
---

# mcpx development

mcpx is a single-binary, single-YAML security gateway for MCP servers (auth,
policy, rate limiting, audit, integrity pinning) plus `mcpx scan`, which checks
locally configured MCP servers for tool poisoning. Its promise — one binary, one
config, secure by default — constrains every change.

## Layout

- `cmd/mcpx/` — CLI entrypoint, middleware chain, `scan` subcommand, version info.
- `internal/<pkg>/` — one concern per package: `auth`, `policy`, `ratelimit`,
  `audit`, `proxy`, `stdio`, `openapi`, `integrity`, `scan`, `config`, `cache`,
  `breaker`, `tracing`, `metrics`, `health`, `cors`, `httperr`, `mcp`, `middleware`.
- `mcpx.yaml` — example config. `deploy/` — Grafana/Prometheus. `examples/` —
  including a deliberately poisoned MCP server for scan tests.
- `.goreleaser.yaml`, `install.sh`, `Dockerfile.release` — release tooling.

## Commands

```bash
go build ./...
go vet ./...
go test -race ./...
golangci-lint run      # CI uses golangci-lint v2.3
goreleaser check       # when touching release tooling
```

Go and golangci-lint must be on `PATH`. If golangci-lint panics with "file
requires newer Go version", the binary was built with an older Go than your
toolchain: run it with `GOTOOLCHAIN=<the Go version it was built with>`, or
reinstall it. That is a tool problem, not a lint failure.

CI (`.github/workflows/ci.yml`) runs build, `go test -race`, vet and
golangci-lint on Go 1.22. Keep code compatible with the `go` directive in
`go.mod`; don't use language or stdlib features newer than it.

## Design principles (ROADMAP.md)

1. **Zero dependencies at runtime** — single static binary, no sidecars, no databases.
2. **One config file** — everything in `mcpx.yaml`, no scattered env vars.
3. **Protocol-aware** — the gateway understands MCP, not just HTTP.
4. **Secure by default** — auth on, audit on, deny-by-default policies.

## Maintainer conventions

These are not in ROADMAP.md but are how the project is run:

- A new Go module dependency needs a written reason in the PR.
- New behavior is a field in `mcpx.yaml`, validated in `internal/config` with a
  clear error message.
- Security features fail closed. A scan that checked nothing is an error, never
  "clean".
- No breaking config changes without a deprecation path.

## Every change also updates

- `CHANGELOG.md` under `## [Unreleased]`, in the right section: Added, Changed,
  Fixed or Security. New features never go under Fixed.
- `README.md` if users see it (config field, CLI flag, scan rule table).
- `ROADMAP.md` checkbox if it completes a roadmap item.
- Tests next to the code (`*_test.go`), table-driven; scan rules get a positive
  and a negative fixture. Timing-based tests must not depend on sleeps alone —
  stress them with `go test -race -count=30 -run <Test>` before pushing.

## Workflow

- Branch from fresh `origin/main`. If the main checkout may hold someone's
  uncommitted work, use a separate git worktree.
- Follow the `adversarial-pr` skill for the PR cycle and `agent-log` for logging.
- Releases: merge the release-prep PR, then a human pushes a `vX.Y.Z` tag; the tag
  triggers `.github/workflows/release.yml`. Agents do not tag or publish.

## Example: adding a scan rule

1. Add the detector in `internal/scan/` with a table test covering a hit, a near
   miss, and hidden-Unicode variants.
2. Add a fixture tool to `examples/poisoned-mcp` if end-to-end coverage helps.
3. Add a row to the README rule table (rule id, severity, what it catches).
4. CHANGELOG: `### Added` — "`mcpx scan`: `<rule-id>` rule detects …".

## Guardrails

- Never weaken a security default to make a test pass.
- Cite the spec or CVE for any security claim in code comments, docs or PRs —
  verify it online first.
- Never push to `main`, tag, or publish.
- Never report a check as passing that you did not run.
