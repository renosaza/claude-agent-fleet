---
name: go-standards
description: Go code standards and toolchain. Use when writing, reviewing, testing, or debugging Go (.go / go.mod) — formatting, linting, type-checking via the compiler, race-aware tests, and dependency vuln scanning for this stack.
---

# go-standards

Tool categories are the standard; concrete tools are current (refreshed by pattern-curator).
Do not hard-pin versions in agent prose.

| Concern | Tool | Command |
|---|---|---|
| package / env | `go` modules | `go mod tidy` |
| format | `gofmt -s` / `gofumpt` | `gofmt -s -w .` (or `gofumpt -w .`) |
| lint | `golangci-lint` v2 (wraps `go vet` + `staticcheck`) | `golangci-lint run` |
| type-check | compiler + `go vet` | `go build ./... && go vet ./...` |
| test | `go test` | `go test ./... -race -cover` |
| vuln scan | `govulncheck` | `govulncheck ./...` |

## Definition of done (run before declaring done)
```
gofmt -s -w . && golangci-lint run && go vet ./... && go test ./... -race -cover
```
Follow the repo's `.golangci.yml` and module layout when present — the repo's settings win over
these defaults. See `references/` for project overrides.
