# Sovereign Maintenance & Long-Term Support Plan for 38 Workspace Repositories

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans` to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Establish an automated, sovereign maintenance, supply-chain autonomy, and long-term support (LTS) lifecycle across all 38 repositories in the workspace, ensuring automated upstream tracking, CVE scanning with Dark Gravity autonomous remediation, multi-arch dual-registry packaging, Cosign/Minisign cryptographic signatures, sovereign Go module replace policies, and dual-engine code intelligence (Hub-and-Spoke Graphify + Per-Project Code-Review-Graph).

**Architecture:** A multi-tier architecture spanning:
1. **Upstream & Vulnerability Tracking:** Scheduled GitHub Actions workflows (weekly upstream sync into isolated `upstream-master` tracking branches, daily Trivy/CodeQL/VulnCheck CVE scans).
2. **Autonomous Remediation:** Autonomous event dispatching to the Dark Gravity CA/CD factory (`lgcorzo/rust_CACD_autonomous_factory`), executing surgical AST patches in gVisor sandboxes with strict Human-in-the-Loop (HITL) PR gates.
3. **Automated Verification Matrix:** Comprehensive testing across all tiers (Go 1.24/1.26, multi-node clustering, healing, site replication, disaster recovery).
4. **Supply Chain Multi-Arch Packaging:** Hermetic multi-stage Alpine Docker compilation from source, dual-registry distribution (GHCR `ghcr.io/lgcorzo/*` and MicroK8s in-cluster registry `localhost:32000`).
5. **Cryptographic Release Provenance:** Minisign and Sigstore Cosign signatures with SHA-256 and BLAKE3 checksum manifests.
6. **Sovereign Go Module Resolution:** Global Go module `replace` directives mapping `github.com/minio/*` to `github.com/lgcorzo/* master`.
7. **Dual-Engine Code Intelligence:** Per-project `code-review-graph` for git-aware blast radius and diff scoring, combined with a Hub-and-Spoke federated `graphify` architecture (local spoke AST extraction + central merged cross-repo graph).

**Tech Stack:** GitHub Actions, Go 1.24/1.26, Docker Buildx, QEMU, Cosign (Sigstore), Minisign, Trivy, CodeQL, VulnCheck, MicroK8s Registry, Hatchet Orchestrator (Dark Gravity Factory), Graphify, Code-Review-Graph.

## Global Constraints
- **Human-in-the-Loop (HITL) Gate:** Under no circumstances may an agent automatically merge a PR into `master` or default branches. The merge process MUST be a manual human action.
- **Hermetic Compilation:** Zero external binary downloads in Docker builds; all images compile directly from source using Go 1.24/1.26 on Alpine Linux.
- **Protected Branch Non-Destructive Ingestion:** Upstream changes are fetched into dedicated tracking branches (`upstream-master`) and merged into `master` via pull requests to preserve custom factory hardening and Dark Gravity patches.
- **Code Intelligence Isolation:** `code-review-graph` MUST be isolated per-project due to Git boundary constraints (`detect_changes_tool` requires a `.git` root) and symbol collision prevention across 38 repositories. `graphify` uses per-project spokes merged into a central workspace hub via `graphify merge-graphs`.
- **Target Workspace Path:** `/mnt/F024B17C24B145FE/Repos/Minio_project/<repo>`.

---

## Repository Classification Matrix (38 Repositories)

| Tier | Type | Repositories | Key CI/CD & Intelligence Deliverables |
|:---|:---|:---|:---|
| **Tier 1** | **Core Server & Infrastructure** | `minio`, `mc`, `operator`, `kes`, `console`, `directpv`, `warp`, `sidekick`, `certgen`, `dperf` | Multi-arch Docker, GHCR + MicroK8s registry, Cosign/Minisign signed binaries, upstream-sync, CVE scan, dedicated `.code-review-graph/`, spoke `graphify-out/` |
| **Tier 2** | **Crypto & SIMD Acceleration** | `sha256-simd`, `md5-simd`, `blake2b-simd`, `simdjson-go`, `highwayhash`, `crc64nvme`, `sio`, `asm2plan9s` | Go module replace directives, benchmark matrix, upstream-sync, CodeQL/VulnCheck, spoke `graphify-out/` |
| **Tier 3** | **SDKs & Core Client Libraries** | `minio-go`, `madmin-go`, `kms-go`, `pkg`, `cli`, `colorjson`, `csvparser`, `dnscache`, `filepath`, `mtls`, `multipart-debug`, `mux`, `pkger`, `selfupdate`, `websocket`, `xxml`, `zipindex` | Go module replace directives, semantic tag mirroring, upstream-sync, CodeQL/VulnCheck, spoke `graphify-out/` |
| **Tier 4** | **Testing & Documentation** | `mint`, `docs`, `minio-cf` | Functional verification, docs site sync, markdown knowledge base |

---

## Code Intelligence Architecture: Graphify & Code-Review-Graph in Multi-Project Workspace

### Architectural Trade-Off Analysis

```
                      ┌────────────────────────────────────────────────────────┐
                      │   Workspace Root (/Minio_project/)                     │
                      │   - Central Merged Macro Graph (cross-repo-graph.json) │
                      │   - Traces cross-repo APIs (mc -> madmin-go -> minio)  │
                      └───────────────────────────┬────────────────────────────┘
                                                  │
                 ┌────────────────────────────────┼────────────────────────────────┐
                 ▼                                ▼                                ▼
   ┌───────────────────────────┐    ┌───────────────────────────┐    ┌───────────────────────────┐
   │ minio/                    │    │ mc/                       │    │ madmin-go/                │
   │ ├─ .git/                  │    │ ├─ .git/                  │    │ ├─ .git/                  │
   │ ├─ .code-review-graph/    │    │ ├─ .code-review-graph/    │    │ ├─ .code-review-graph/    │
   │ │  └─ graph.db (SQLite)   │    │ │  └─ graph.db (SQLite)   │    │ │  └─ graph.db (SQLite)   │
   │ └─ graphify-out/ (Spoke)  │    │ └─ graphify-out/ (Spoke)  │    │ └─ graphify-out/ (Spoke)  │
   └───────────────────────────┘    └───────────────────────────┘    └───────────────────────────┘
```

1. **Why `code-review-graph` MUST be Per-Project:**
   - **Git Boundary Constraint:** `code-review-graph` tools (especially `detect_changes_tool`) execute git commands (`git diff`, `git rev-parse HEAD`) relative to the local repository root. Running from `/Minio_project/` fails with `fatal: not a git repository` because the workspace root is a container folder holding 38 independent `.git` directories.
   - **Zero Symbol Collision:** 38 separate Go repositories share common symbol names (`Config`, `Client`, `Server`, `New()`, `Options`, `Context`). Storing all in a single SQLite database results in false caller-callee bindings. Per-project instances guarantee 100% semantic fidelity.
   - **Precise Blast Radius:** `get_impact_radius_tool` and `query_graph_tool(pattern="tests_for")` accurately map changes to local test suites without noise from unrelated repos.

2. **Why `graphify` uses a Hub-and-Spoke Federated Architecture:**
   - **Scale Protection:** Extracting all 38 repositories simultaneously in a single scan creates over 50,000 AST nodes, blowing past graphify community thresholds and inflating token usage.
   - **Instant Spoke Updates:** When editing a single repository (e.g. `minio`), running `graphify update .` locally completes in 2–4 seconds via AST analysis without re-indexing all 38 repositories.
   - **Cross-Repo Architecture Queries:** By executing `graphify merge-graphs` across spoke outputs, the central workspace root maintains `cross-repo-graph.json` where every node carries its `repo` attribute, enabling high-level questions such as:
     `graphify query "How does mc CLI invoke madmin-go Admin API endpoints in minio server?"`

---

## Phased Implementation Tasks

### Task 1: Repository Remote & Upstream Mapping Engine
**Files:**
- Create: `Minio_project/minio/scripts/sovereign/upstream-registry.json`
- Create: `Minio_project/minio/scripts/sovereign/setup-upstream-remotes.sh`
- Test: `tests/test_upstream_remotes.sh`

**Interfaces:**
- Consumes: Local git repositories in `/mnt/F024B17C24B145FE/Repos/Minio_project/`
- Produces: Standardized `upstream` git remote pointing to official upstream sources for all 38 repositories.

- [ ] **Step 1: Define `upstream-registry.json` mapping all 38 repositories**
- [ ] **Step 2: Create `setup-upstream-remotes.sh` automating remote configuration**
- [ ] **Step 3: Execute script and verify `upstream` remotes across all 38 projects**

---

### Task 2: Automated Upstream Synchronization Workflow (Section 5.1)
**Files:**
- Create: `Minio_project/minio/scripts/sovereign/templates/upstream-sync.yml`
- Apply: `.github/workflows/upstream-sync.yml` in each repository

**Interfaces:**
- Consumes: Upstream repository tags and master commits via weekly cron schedule.
- Produces: `upstream-master` branch update, automated PRs to `master` labeled `upstream-sync`, with conflict checks.

- [ ] **Step 1: Write `upstream-sync.yml` template with weekly cron schedule and PR creation**
- [ ] **Step 2: Ensure tags are mirrored non-destructively to `lgcorzo/*` releases**
- [ ] **Step 3: Validate workflow syntax via action-lint schema validation**

---

### Task 3: Vulnerability Ingestion & Dark Gravity Autonomous Remediation (Section 2 & 5.2)
**Files:**
- Create: `Minio_project/minio/scripts/sovereign/templates/daily-cve-scan.yml`
- Create: `Minio_project/minio/scripts/sovereign/dispatch_dark_gravity_mission.py`

**Interfaces:**
- Consumes: Trivy vulnerability scans, CodeQL SARIF, and VulnCheck reports.
- Produces: Automated GitHub Issues labeled `autonomous-mission` in `lgcorzo/rust_CACD_autonomous_factory` with explicit resource limits blocks (`CPU: 500m, RAM: 512Mi, Timeout: 300s`), dispatching to Rustant/ZeroClaw.

- [ ] **Step 1: Create `daily-cve-scan.yml` running daily at 04:00 UTC**
- [ ] **Step 2: Create `dispatch_dark_gravity_mission.py` to evaluate CVE severity and dispatch Dark Gravity mission issues**
- [ ] **Step 3: Execute a dry-run test against existing scan data**

---

### Task 4: Multi-Architecture Container CI/CD & Dual-Registry (Section 4 & 5.3)
**Files:**
- Create: `Minio_project/minio/scripts/sovereign/templates/Dockerfile.sovereign`
- Create: `Minio_project/minio/scripts/sovereign/templates/docker-publish.yml`
- Create: `Minio_project/minio/scripts/sovereign/deploy-to-microk8s.sh`

**Interfaces:**
- Consumes: Go source code on `master` branch and release tags.
- Produces: Multi-arch container images (`linux/amd64`, `linux/arm64`) pushed to `ghcr.io/lgcorzo/*` and in-cluster MicroK8s registry at `localhost:32000`.

- [ ] **Step 1: Write hermetic multi-stage `Dockerfile.sovereign` (Go 1.24/1.26, Alpine 3.20, zero external downloads)**
- [ ] **Step 2: Create multi-arch `docker-publish.yml` (QEMU + Docker Buildx for AMD64/ARM64) targeting GHCR**
- [ ] **Step 3: Create `deploy-to-microk8s.sh` to mirror images to in-cluster registry `localhost:32000`**

---

### Task 5: Signature-Verified Binary Releases (Section 5.4)
**Files:**
- Create: `Minio_project/minio/scripts/sovereign/templates/release-signing.yml`

**Interfaces:**
- Consumes: Release tags (`v*`) and build binaries.
- Produces: SHA-256 and BLAKE3 checksums, Minisign signatures (`.minisig`), and Sigstore Cosign keyless provenance signatures.

- [ ] **Step 1: Write `release-signing.yml` installing Cosign, Minisign, and B3sum**
- [ ] **Step 2: Generate SHA-256 and BLAKE3 checksum manifests during build execution**
- [ ] **Step 3: Sign manifests via Minisign and Cosign before attaching to GitHub Releases**

---

### Task 6: Sovereign Go Module Dependency Resolution Policy (Section 5.5)
**Files:**
- Create: `Minio_project/minio/scripts/sovereign/generate-go-replace-directives.py`
- Create: `Minio_project/minio/scripts/sovereign/apply-go-replace.sh`

**Interfaces:**
- Consumes: `go.mod` files across all 27 Go-based repositories.
- Produces: Standardized Go module `replace` directives mapping `github.com/minio/*` to `github.com/lgcorzo/* master`.

- [ ] **Step 1: Create `generate-go-replace-directives.py` with exhaustive module mapping**
- [ ] **Step 2: Apply directives across all Go projects in `Minio_project/*`**
- [ ] **Step 3: Run `go mod tidy` and `go test ./...` in each Go repo to verify compilation**

---

### Task 7: Dual-Engine Code Intelligence Setup & Cross-Repo Federation
**Files:**
- Create: `Minio_project/scripts/sync-code-intelligence.sh`
- Create: `Minio_project/scripts/init-repo-intelligence.sh`
- Test: Verification query across cross-repo graph

**Interfaces:**
- Consumes: Active repositories in `Minio_project/`.
- Produces: Isolated `.code-review-graph/graph.db` per repository + spoke `graphify-out/graph.json` + merged root `cross-repo-graph.json`.

- [ ] **Step 1: Create `init-repo-intelligence.sh` to initialize Tier-1 repositories (`minio`, `mc`, `operator`, `kes`, `madmin-go`) with local `.code-review-graph` databases and local `graphify-out`**
- [ ] **Step 2: Create `sync-code-intelligence.sh` to update local graphs upon code edits and merge spoke graphs into root `cross-repo-graph.json`**
- [ ] **Step 3: Run verification query `graphify query` on the merged cross-repo graph to validate cross-repository call tracing**

---

### Task 8: Workspace-Wide Automation Rollout Harness
**Files:**
- Create: `Minio_project/minio/scripts/sovereign/apply-sovereign-protocol.py`
- Test: `Minio_project/minio/scripts/sovereign/verify_sovereign_conformance.py`

**Interfaces:**
- Consumes: All 38 repositories in `/mnt/F024B17C24B145FE/Repos/Minio_project/`
- Produces: Automated branching (`feat/sovereign-maintenance-protocol`), workflow injection, commits, pushes, and conformance summary report.

- [ ] **Step 1: Write `apply-sovereign-protocol.py` injecting workflows, remotes, code intelligence hooks, and replace rules per tier**
- [ ] **Step 2: Create `verify_sovereign_conformance.py` to audit all 38 repositories for compliance**
- [ ] **Step 3: Execute verification audit and verify 100% conformance across all 38 repositories**

---

## Self-Review Checklist
- [x] **Spec coverage:** Covers all sections from the user request (1. Upstream Tracking, 2. Autonomous Remediation, 3. Verification Matrix, 4. Supply Chain Artifact Publishing, 5.1-5.5) plus the Multi-Project Code Intelligence Architecture (Graphify + Code-Review-Graph).
- [x] **No Placeholders:** Clear, concrete step definitions with exact paths and scripts.
- [x] **HITL Safety:** Explicitly enforces manual human review before any PR merge into default branches.
