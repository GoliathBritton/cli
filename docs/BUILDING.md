# Building `gh` (local & CI guidance)

This file contains recommended commands for building `gh` locally and in CI. It also includes safe workarounds for module checksum issues.

Local quick build (developer):

```bash
# clear local mod cache to force fresh downloads
go clean -modcache

# Build the local binary (preferred for dev)
GOPROXY=direct go build -v -o gh ./cmd/gh

# or install in your $GOBIN
GOPROXY=direct go install -v github.com/cli/cli/v2/cmd/gh@latest
```

Notes:
- Use `GOPROXY=direct` to bypass intermediate proxies when diagnosing checksum issues.
- `GOSUMDB=sum.golang.org` is the default trusted checksum DB; do not disable it unless you understand the security implications.

Vendor mode (recommended for reproducible CI builds):

```bash
# create a reproducible vendor/ directory (run locally, commit vendor/ to repo if desired for CI)
go mod vendor

# build using vendor mode in CI: (ensures builds don't rely on network)
GOFLAGS=-mod=vendor go build -v ./cmd/gh
```

CI recommendations (GitHub Actions):
- Use the workflow `.github/workflows/go-build.yml` in this repo. It:
  - sets Go 1.24
  - caches module downloads and build cache
  - prefers vendor mode when `vendor/` exists
  - otherwise uses `GOPROXY=https://proxy.golang.org,direct` and `GOSUMDB=sum.golang.org`

Diagnosing checksum mismatches
--------------------------------
If you see a checksum mismatch like:

  SECURITY ERROR
  downloaded: h1:...\n  sum.golang.org: h1:...

Run these commands and paste outputs when filing an upstream issue or asking for help:

```bash
go env GOPROXY GOSUMDB GOPRIVATE GONOSUMDB
go clean -modcache
GOPROXY=direct go mod download -json github.com/charmbracelet/glamour@v0.10.0
```

If `GOPROXY=direct` still fails and reports a checksum different from `sum.golang.org`, collect the two hashes and either:
- open an issue against the upstream module (example: `github.com/charmbracelet/glamour`), or
- contact your corporate GOPROXY operator if you are using one.
