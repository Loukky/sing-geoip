# sing-geoip

Single-package Go tool that converts MaxMind GeoIP data from `Loukky/geoip` releases into sing-box compatible MMDB and SRS rule-set files.

## Commands

```sh
# Run (downloads upstream GeoIP, generates outputs)
go run -v .

# Test
go test -v ./...

# Format (gofumpt + gofmt -s + gci with sagernet import ordering)
make fmt

# Lint
make lint
```

## Build output

- `geoip.db` — full database
- `geoip-cn.db` — CN-only subset
- `rule-set/geoip-*.srs` — per-country rule sets

Artifacts in `.gitignore`: `*.db`, `*.mmdb`, `/rule-set/`, `/release/`.

## Key env vars

| Var | Effect |
|---|---|---|
| `ACCESS_TOKEN` | GitHub API auth (optional, for higher rate limits) |
| `FIXED_RELEASE` | Pin a specific upstream release tag instead of latest |
| `NO_SKIP=true` | Force rebuild even if destination release is up-to-date |
| `TAG_NAME` | Override release tag (default: `date +%Y%m%d%H%M` from workflow) |

## CI

- **build.yaml** — runs on push to `main`; builds with `NO_SKIP=true`
- **release.yaml** — runs on schedule (12th monthly) or `workflow_dispatch`; pushes to `release` and `rule-set` branches + creates GitHub release

## Release flow

1. Workflow generates `TAG_NAME=$(date +%Y%m%d%H%M)`
2. `go run -v .` downloads `Country.mmdb` from `Loukky/geoip` latest release, converts it
3. If destination (`sagernet/sing-geoip`) release tag already contains the source tag, it skips (unless `NO_SKIP=true`)
4. On release: creates GitHub release with `TAG_NAME`, pushes `release` branch with `.db`+`.sha256sum`, `rule-set` branch with `.srs` files
5. Old releases/tags cleaned up (keeps latest 3, `delete_tags: true`)

## Import ordering (gci)

Standard imports → `github.com/sagernet/*` → everything else. Applied by `make fmt` and enforced by `golangci-lint`.
