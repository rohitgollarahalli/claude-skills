---
name: mcpx-dev
description: Build, test, lint, document and release changes to mcpx, the Go MCP security gateway (github.com/rohitgollarahalli/mcpx). Use for any work in the mcpx repo — features, fixes, `mcpx scan` rules, config fields, transports, release prep — or when the user mentions mcpx, the gateway, or `mcpx scan`. Fires even if the user does not name the skill.
---

# mcpx development

mcpx is a single-binary, single-YAML security gateway for MCP servers (auth,
policy, rate limiting, audit, integrity pinning) plus `mcpx scan`, which checks
locally configured MCP servers for tool poisoning. Its promise — one binary, one
config, secure by default — constrains every change.

## Layout

- `cmd/mcpx/` — CLI entrypoint, middleware chain, `scan` subcommand.
- `internal/<pkg>/` — one concern per package: `auth`, `policy`, `ratelimit`,
  `audit`, `proxy`, `stdio`, `openapi`, `integrity`, `scan`, `config`, `cache`,
  `breaker`, `tracing`, `metrics`, `health`, `cors`, `httperr`, `mcp`, `middleware`.
- `mcpx.yaml` — example config. `deploy/` — Grafana/Prometheus. `examples/` —
  including a deliberately poisoned MCP server for scan tests.
- `.goreleaser.yaml`, `install.sh`, `Dockerfile.release` — release tooling.

## Commands

```bash
export PATH=$HOME/.local/go/bin:$HOME/go/bin:$PATH
go build ./...
go vet ./...
go test -race ./...
GOTOOLCHAIN=go1.24.5 golangci-lint run   # local v2.3.1 binary panics on newer Go
goreleaser check                          # when touching release tooling
```

CI (`.github/workflows/ci.yml`) runs build, `go test -race`, vet and
golangci-lint v2.3 on Go 1.22. Keep code compatible with the `go` directive in
`go.mod`; don't use language or stdlib features newer than it.

## Design rules (from ROADMAP.md)

1. **Zero runtime dependencies** — no sidecars, databases, or new heavy modules.
   A new dependency needs a strong written reason in the PR.
2. **One config file** — new behavior is a field in `mcpx.yaml`, validated in
   `internal/config` with a clear error. No env-var-only switches.
3. **Protocol-aware** — inspect MCP JSON-RPC, not just HTTP.
4. **Secure by default** — new features fail closed; a scan that checked nothing
   is an error, never "clean".
5. **No breaking config changes** without a deprecation path.

## Every change also updates

- `CHANGELOG.md` under `## [Unreleased]`, in the right Added/Changed/Fixed section.
- `README.md` if users see it (config field, CLI flag, scan rule table).
- `ROADMAP.md` checkbox if it completes a roadmap item.
- Tests next to the code (`*_test.go`), table-driven; scan rules get a positive
  and a negative fixture.

## Workflow

- Work in a worktree under `~/agent/worktrees/<branch>`, branched from fresh
  `origin/main`; the main `~/mcpx` checkout may hold uncommitted work.
- Follow the `adversarial-pr` skill for the PR cycle and `agent-log` for logging.
- Releases: merge the release PR, then a human tags `vX.Y.Z`; the tag triggers
  `.github/workflows/release.yml`. Agents do not tag.

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
