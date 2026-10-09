# Comprehensive Ecosystem Modernization & Upgrade Plan (38 Workspace Repositories)

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans` to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Modernize, secure, harmonize, and standardize the CI/CD pipelines, runtime dependencies, Go toolchains, dead URL endpoints, container registries, and code intelligence across all 38 repositories in the MinIO project workspace.

**Architecture:** A tiered rollout strategy across 4 architectural layers:
1. **Tier 1 (Foundation & SIMD Primitives):** SIMD accelerators and utility libraries (`sha256-simd`, `md5-simd`, `highwayhash`, `crc64nvme`, `simdjson-go`, `sio`, `asm2plan9s`, `blake2b-simd`, `filepath`, `colorjson`, `csvparser`, `xxml`, `zipindex`).
2. **Tier 2 (Core SDKs & Communication):** Go client libraries and networking (`cli`, `dnscache`, `mtls`, `multipart-debug`, `mux`, `pkger`, `selfupdate`, `websocket`, `minio-go`, `madmin-go`, `kms-go`, `pkg`).
3. **Tier 3 (Core Daemons & Applications):** Mission-critical services and clients (`minio`, `mc`, `kes`, `console`, `operator`, `directpv`, `warp`, `sidekick`, `certgen`, `dperf`).
4. **Tier 4 (Documentation, Packaging & Testing):** E2E harnesses and docs (`docs`, `mint`, `minio-cf`).

**Tech Stack:** Go 1.24/1.26, GitHub Actions, Docker / GHCR (`ghcr.io/lgcorzo/*`), NodeJS / Yarn, Code-Review-Graph, Graphify.

## Global Constraints
- **Autonomous Sovereign Boundaries:** All external links pointing to archived `dl.min.io` endpoints MUST be replaced with direct hermetic extraction from sovereign containers or GitHub Releases.
- **AIStor Extension Naming Convention:** Call any non-AWS action, API, or behavior an **AIStor extension**, never a "MinIO extension".
- **Container Registry Standardization:** Replace dead/unauthorized `quay.io/minio/*` references in test fixtures and workflows with sovereign `ghcr.io/lgcorzo/*` images.
- **Go Toolchain Matrix:** Minimum supported Go version floor is `1.22+` for low-level SIMD primitives, `1.24.4` for standard SDKs and tools, and `1.26.0` for high-performance servers (`minio`, `pkg`, `directpv`).
- **GitHub Actions Runners:** Standardize GitHub Action actions to modern releases: `actions/checkout@v4`, `actions/setup-go@v5`, `actions/setup-node@v4`, `actions/cache@v4`. Eliminate all Node.js 20 deprecated action invocations.
- **Human-in-the-Loop (HITL) Gate:** Changes to `main`/`master` must be submitted via branch and verified via GitHub Actions CI prior to merging.

---

## Repository Classification & Dependency Waves

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│ WAVE 1: SIMD, Math & Data Primitives (No internal Go dependencies)             │
│ sha256-simd, md5-simd, highwayhash, crc64nvme, simdjson-go, sio, asm2plan9s,    │
│ blake2b-simd, filepath, colorjson, csvparser, xxml, zipindex                   │
└───────────────────────────────────────┬─────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ WAVE 2: SDKs, Middleware & Drivers (Depend on Wave 1)                          │
│ cli, dnscache, mtls, multipart-debug, mux, pkger, selfupdate, websocket,        │
│ minio-go, madmin-go, kms-go, pkg                                                │
└───────────────────────────────────────┬─────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ WAVE 3: Core Servers, Controllers & Clients (Depend on Waves 1 & 2)             │
│ minio, mc, kes, console, operator, directpv, warp, sidekick, certgen, dperf     │
└───────────────────────────────────────┬─────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ WAVE 4: Documentation, Integration Harnesses & Packaging                        │
│ docs, mint, minio-cf                                                            │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Detailed Implementation Tasks

### Task 1: Audit & Replace Dead `dl.min.io` URLs Across All Repositories

**Files:**
- Modify: `minio/Makefile:55-75`
- Modify: `minio/docs/site-replication/*.sh`
- Modify: `minio-go/.github/workflows/go-windows.yml:32-45`
- Modify: `operator/testing/install-mc.sh:10-35`
- Modify: `operator/testing/common.sh:85-110`
- Modify: `warp/.github/workflows/qreleaser-test.yml:25-40`
- Modify: `console/web-app/tests/scripts/initialize-env.sh:18-35`
- Modify: `console/web-app/tests/scripts/operator.sh:22-40`

**Interfaces:**
- Replaces dead HTTP 410 URLs (`https://dl.min.io/client/mc/release/...`) with container image extraction (`ghcr.io/lgcorzo/mc:latest` or `quay.io/minio/aistor/mc:latest`) or GitHub Releases API.

- [ ] **Step 1: Write verification script to identify all `dl.min.io` instances**
```bash
grep -rn "dl.min.io" /mnt/F024B17C24B145FE/Repos/Minio_project/ --exclude-dir=".git"
```

- [ ] **Step 2: Update `operator/testing/install-mc.sh` and `common.sh`**
Replace curl/wget downloads from dl.min.io with container extraction:
```bash
docker run --rm --entrypoint cat ghcr.io/lgcorzo/mc:latest /usr/bin/mc > /tmp/mc
chmod +x /tmp/mc && sudo mv /tmp/mc /usr/local/bin/mc
```

- [ ] **Step 3: Update `minio-go/.github/workflows/go-windows.yml`**
Replace download URLs with official GitHub Release release assets:
```yaml
run: |
  curl.exe -L -o minio.exe https://github.com/lgcorzo/minio/releases/latest/download/minio.exe
```

- [ ] **Step 4: Update `minio/Makefile` and replication test scripts**
Replace curl invocation in `minio/Makefile` with release asset fallback or container binary extraction.

- [ ] **Step 5: Run tests and verify zero `dl.min.io` references remain**
Run: `grep -rn "dl.min.io" /mnt/F024B17C24B145FE/Repos/Minio_project/ --exclude-dir=".git" | wc -l`
Expected: `0`

---

### Task 2: Standardize GitHub Actions Workflows to Modern v4/v5 Actions

**Files:**
- Modify: `cli/.github/workflows/go.yml`
- Modify: `crc64nvme/.github/workflows/*.yml`
- Modify: `csvparser/.github/workflows/*.yml`
- Modify: `filepath/.github/workflows/*.yml`
- Modify: `highwayhash/.github/workflows/*.yml`
- Modify: `md5-simd/.github/workflows/*.yml`
- Modify: `mint/.github/workflows/*.yml`
- Modify: `mux/.github/workflows/*.yml`
- Modify: `operator/.github/workflows/*.yml`
- Modify: `sha256-simd/.github/workflows/*.yml`
- Modify: `websocket/.github/workflows/*.yml`
- Modify: `xxml/.github/workflows/*.yml`
- Modify: `zipindex/.github/workflows/*.yml`

**Interfaces:**
- Upgrades `actions/checkout@v1..v3` -> `actions/checkout@v4`
- Upgrades `actions/setup-go@v1..v4` -> `actions/setup-go@v5`
- Upgrades `actions/cache@v1..v3` -> `actions/cache@v4`
- Adds `cache: true` where applicable to accelerate CI times.

- [ ] **Step 1: Write automated workflow updater script**
```bash
python3 -c "
import os, glob

base = '/mnt/F024B17C24B145FE/Repos/Minio_project'
for r in os.listdir(base):
    for wf in glob.glob(os.path.join(base, r, '.github', 'workflows', '*.y*ml')):
        with open(wf, 'r') as f:
            c = f.read()
        c2 = c.replace('actions/checkout@v2', 'actions/checkout@v4')
        c2 = c2.replace('actions/checkout@v3', 'actions/checkout@v4')
        c2 = c2.replace('actions/setup-go@v1', 'actions/setup-go@v5')
        c2 = c2.replace('actions/setup-go@v2', 'actions/setup-go@v5')
        c2 = c2.replace('actions/setup-go@v3', 'actions/setup-go@v5')
        c2 = c2.replace('actions/setup-go@v4', 'actions/setup-go@v5')
        c2 = c2.replace('actions/cache@v1', 'actions/cache@v4')
        c2 = c2.replace('actions/cache@v2', 'actions/cache@v4')
        c2 = c2.replace('actions/cache@v3', 'actions/cache@v4')
        if c2 != c:
            with open(wf, 'w') as f:
                f.write(c2)
            print(f'Updated {wf}')
"
```

- [ ] **Step 2: Inspect workflow diffs across Wave 1 & 2 repositories**
Run: `git diff .github/workflows` across updated repositories.
Expected: Clean upgrade to `@v4` and `@v5`.

- [ ] **Step 3: Validate syntax with `yamllint` or action linters**
Run local syntax validation across all updated YAML files.

---

### Task 3: Establish Missing CI Workflows in Repositories Without GitHub Actions

**Repositories without workflows:**
- `asm2plan9s`, `blake2b-simd`, `dnscache`, `minio-cf`, `mtls`, `multipart-debug`

**Files:**
- Create: `asm2plan9s/.github/workflows/go.yml`
- Create: `blake2b-simd/.github/workflows/go.yml`
- Create: `dnscache/.github/workflows/go.yml`
- Create: `mtls/.github/workflows/go.yml`
- Create: `multipart-debug/.github/workflows/go.yml`

**Interfaces:**
- Standard multi-OS matrix (`ubuntu-latest`, `macos-latest`, `windows-latest`)
- Go matrix: `['1.24.x', '1.25.x', '1.26.x']`
- Govulncheck security scanning step.

- [ ] **Step 1: Create standard Go CI workflow template**
```yaml
name: CI

on:
  push:
    branches: [ master, main ]
  pull_request:
    branches: [ master, main ]

jobs:
  test:
    name: Test (Go ${{ matrix.go-version }}, ${{ matrix.os }})
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        go-version: [ '1.24.x', '1.25.x' ]
        os: [ ubuntu-latest ]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: ${{ matrix.go-version }}
          cache: true
      - name: Test
        run: go test -v -race ./...
```

- [ ] **Step 2: Add `govulncheck` workflow to repositories**
```yaml
name: Security Scan

on:
  schedule:
    - cron: '0 4 * * *'
  workflow_dispatch:

jobs:
  vulncheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.25.x'
      - name: Run Govulncheck
        run: |
          go install golang.org/x/vuln/cmd/govulncheck@latest
          govulncheck ./...
```

- [ ] **Step 3: Test locally with `go test ./...` in each repository**
Verify all packages pass their tests locally before pushing workflow changes.

---

### Task 4: Harmonize Go Module Dependencies & Sovereign Repositories

**Files:**
- Modify: `madmin-go/go.mod`
- Modify: `sidekick/go.mod`
- Modify: `multipart-debug/go.mod`
- Modify: `warp/go.mod`
- Modify: `operator/go.mod`
- Modify: `colorjson/go.mod`
- Modify: `dperf/go.mod`

**Problem Statement:**
Several intermediate SDKs still reference `github.com/minio/*` directly while downstream apps (`minio`, `mc`, `console`) reference `github.com/lgcorzo/*`. This creates split module graphs and type mismatch errors.

- [ ] **Step 1: Map out module replacement table**
Define canonical sovereign release tags across:
  - `github.com/lgcorzo/cli -> v1.24.2-lgcorzo.1`
  - `github.com/lgcorzo/minio-go/v7 -> v7.0.91-lgcorzo.2`
  - `github.com/lgcorzo/madmin-go/v3 -> v3.0.109-lgcorzo.1`
  - `github.com/lgcorzo/madmin-go/v4 -> v4.6.7-lgcorzo.1`
  - `github.com/lgcorzo/pkg/v3 -> v3.4.0-lgcorzo.1`

- [ ] **Step 2: Update `replace` directives in consumers**
Update `go.mod` in `operator`, `warp`, `sidekick`, and `dperf` with standard sovereign replace blocks:
```go
replace (
	github.com/minio/cli => github.com/lgcorzo/cli v1.24.2-lgcorzo.1
	github.com/minio/minio-go/v7 => github.com/lgcorzo/minio-go/v7 v7.0.91-lgcorzo.2
	github.com/minio/pkg/v3 => github.com/lgcorzo/pkg/v3 v3.4.0-lgcorzo.1
)
```

- [ ] **Step 3: Run `go mod tidy` and verify compilation**
Run: `go test ./...` in each updated repository.
Expected: Clean pass with no module conflict warnings.

---

### Task 5: Upgrade Container Registry References to Sovereign `ghcr.io/lgcorzo/*`

**Files:**
- Modify: `operator/testing/*.sh`
- Modify: `directpv/pkg/admin/installer/*.go`
- Modify: `console/web-app/tests/scripts/*.sh`
- Modify: `minio/.github/workflows/docker-publish.yml`
- Modify: `mc/.github/workflows/docker-publish.yml`

**Interfaces:**
- Replaces legacy/commercial `quay.io/minio/*` and `quay.io/minio/aistor/*` references with `ghcr.io/lgcorzo/*`.
- Standardizes container image tags on:
  - `ghcr.io/lgcorzo/minio:latest`
  - `ghcr.io/lgcorzo/mc:latest`
  - `ghcr.io/lgcorzo/operator:latest`
  - `ghcr.io/lgcorzo/console:latest`
  - `ghcr.io/lgcorzo/directpv:latest`
  - `ghcr.io/lgcorzo/kes:latest`

- [ ] **Step 1: Update container image defaults in `directpv/pkg/admin/installer/args.go`**
Ensure default sidecar and container registries point to accessible public images.

- [ ] **Step 2: Update test harnesses in `operator` and `console`**
Verify all integration test scripts use `ghcr.io/lgcorzo/minio:latest` and `ghcr.io/lgcorzo/mc:latest`.

- [ ] **Step 3: Validate Docker builds**
Ensure multi-arch Dockerfiles compile locally without network errors:
```bash
docker build -t ghcr.io/lgcorzo/minio:test minio/
```

---

### Task 6: Deploy Code Intelligence (Code-Review-Graph & Graphify)

**Files:**
- Create: `.code-review-graph/` in each of the 38 repositories.
- Create: `graphify-out/` spoke graphs in Tier 1 and Tier 2 repositories.
- Create: Merged macro architecture graph at workspace root.

**Interfaces:**
- `code-review-graph`: provides Git-aware blast radius analysis and semantic code search.
- `graphify`: provides cross-repo visual architecture graphs, dependency flow maps, and component clustering.

- [ ] **Step 1: Initialize per-project `code-review-graph` indices**
Run across all 38 repositories:
```bash
python3 -c "
import os, subprocess

base = '/mnt/F024B17C24B145FE/Repos/Minio_project'
for r in sorted(os.listdir(base)):
    p = os.path.join(base, r)
    if os.path.isdir(os.path.join(p, '.git')):
        print(f'Initializing code-review-graph for {r}...')
        subprocess.run(['code-review-graph', 'index'], cwd=p, check=False)
"
```

- [ ] **Step 2: Run spoke Graphify analysis for Tier 1 repositories**
Generate spoke knowledge graphs for `minio`, `mc`, `operator`, `console`, `directpv`, `kes`.

- [ ] **Step 3: Merge spoke graphs into workspace hub graph**
Merge spoke graphs into `/mnt/F024B17C24B145FE/Repos/Minio_project/graphify-out/` to enable unified cross-repo dependency exploration.

---

## Execution Verification Matrix

| Wave | Repositories | Validation Command | Success Criteria |
| :--- | :--- | :--- | :--- |
| **Wave 1** | SIMD & Primitives | `go test -v -race ./...` | 100% unit tests green, no legacy GHA actions |
| **Wave 2** | SDKs & Drivers | `go test -v -race ./...` | All sovereign module tags resolved cleanly |
| **Wave 3** | Core Servers | `make test` / `make test-race` | Zero 410 dead URLs, zero Quay license errors |
| **Wave 4** | Docs & Harnesses | `npm test` / `make html` | Clean doc builds, zero dead links |

---

## Plan Status & Handoff

Plan complete and saved to `/mnt/F024B17C24B145FE/Repos/Minio_project/docs/plans/2026-10-09-ecosystem-modernization-and-upgrade-plan.md`.
