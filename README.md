# Sovereign Documentation Engine (@lgcorzo/docs)

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-blue.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Sovereign Ecosystem](https://img.shields.io/badge/Sovereign_Ecosystem-@lgcorzo-blueviolet.svg)](https://github.com/lgcorzo)
[![Build Status](https://github.com/lgcorzo/docs/actions/workflows/makefile.yml/badge.svg)](https://github.com/lgcorzo/docs/actions)

This repository forms the central documentation build engine and architectural reference source for the **Sovereign MinIO Ecosystem** under `@lgcorzo`. It powers static site generation, API reference compilation, and deployment guides across all core components of the autonomous storage factory.

---

## Dark Gravity Factory & Sovereign Maintenance Rationale

This repository is actively maintained under `@lgcorzo` as a critical pillar of the **Dark Gravity Factory** autonomous AI production ecosystem.

### Key Maintenance Pillars

1. **Full Supply-Chain Autonomy:**
   - Zero reliance on upstream breaking license changes, deprecation notices, or unannounced URL host retirements.
   - Independent verification, hermetic build environments, and complete ownership of docs build toolchains and rendering scripts.

2. **Dark Gravity Factory Core Integration:**
   - Serves as the central knowledge engine powering autonomous agent pipelines, developer documentation, deployment runbooks, and architectural specifications across high-throughput storage and cryptographic security modules.

3. **Compliance & Security Standards:**
   - Maintained under strict SLA compliance aligned with **EU AI Act**, **SOC 2 Type II**, and **ISO 25059** requirements.
   - Continuous security posture with zero-CVE SLAs enforced through automated daily vulnerability scanning (`Trivy`, `CodeQL`, `govulncheck`).

4. **Ecosystem Interoperability:**
   - Seamless cross-reference integration across all 38 repositories in the `@lgcorzo` sovereign ecosystem (MinIO Server, MC CLI, KES, Operator, DirectPV, Console, SIMD libraries, and client SDKs).

---

## Sovereign Ecosystem Architecture & Repositories (38 Repositories)

> For full repository matrix, live release tags, Dependabot status, and cluster deployment specs, see **[SOVEREIGN_ECOSYSTEM_STATUS.md](SOVEREIGN_ECOSYSTEM_STATUS.md)**.

The following table summarizes the 38 interconnected repositories maintained under the `@lgcorzo` sovereign umbrella:

| Tier | Component Type | Repositories | Role in Dark Gravity Factory |
|:---|:---|:---|:---|
| **Tier 1** | **Core Storage & Infrastructure** | `minio`, `mc`, `operator`, `kes`, `console`, `directpv`, `warp`, `sidekick`, `certgen`, `dperf` | High-throughput object storage, K8s orchestration, cryptographic KMS, web console, bare-metal storage provisioning, and performance benchmarking. |
| **Tier 2** | **Crypto & SIMD Acceleration** | `sha256-simd`, `md5-simd`, `blake2b-simd`, `simdjson-go`, `highwayhash`, `crc64nvme`, `sio`, `asm2plan9s` | Hardware-accelerated cryptographic primitives, SIMD JSON parsing, and streaming encryption for maximum throughput. |
| **Tier 3** | **SDKs & Core Client Libraries** | `minio-go`, `madmin-go`, `kms-go`, `pkg`, `cli`, `colorjson`, `csvparser`, `dnscache`, `filepath`, `mtls`, `multipart-debug`, `mux`, `pkger`, `selfupdate`, `websocket`, `xxml`, `zipindex` | Multi-language SDKs, administrative management APIs, KMS protocols, and low-level utility libraries for sovereign services. |
| **Tier 4** | **Testing & Documentation** | `mint`, `docs`, `minio-cf` | Automated integration testing suites, Sphinx/Markdown documentation engines, and cloud template configurations. |

### Automated CI/CD & Sovereign Maintenance Lifecycle

```
┌────────────────────────────────────────────────────────────────────────┐
│                      Upstream Tracking Branch                          │
│                     (weekly upstream-sync cron)                        │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   │
                                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│                   Dark Gravity Factory CA/CD                           │
│     (Daily Trivy / CodeQL / VulnCheck Scans + Autonomous AST Patching) │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   │
                                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│                  Human-in-the-Loop (HITL) Gate                         │
│             (Mandatory code review & sign-off prior to merge)          │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   │
                                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│                  Hermetic Multi-Stage Build & Release                  │
│       (Multi-arch AMD64/ARM64, GHCR + Local Registry, Minisign/Cosign) │
└────────────────────────────────────────────────────────────────────────┘
```

---

## Build Instructions

`@lgcorzo/docs` uses [Sphinx](https://www.sphinx-doc.org/en/master/index.html) to generate static HTML pages using ReSTructured Text (rST) and Markdown.

### Prerequisites

- Any GNU/Linux Operating System, or macOS 12.3 or later.
- Python 3.10.x or later and `pip`
- `python3-venv`
- Node.js 18+ and `npm`
- `git`

### Local Build Steps

1. Clone docs repository locally:

```bash
git clone https://github.com/lgcorzo/docs && cd docs/
```

2. Create and activate a Python virtual environment:

```bash
python3 -m venv venv && source venv/bin/activate
```

3. Install Python and Node.js dependencies:

```bash
pip install -r requirements.txt && npm install && npm run build
```

4. Compile documentation:

```bash
make SYNC_SDK=true mindocs
```

5. Preview the generated documentation at `http://localhost:8000`:

```bash
python3 -m http.server --directory build/$(git rev-parse --abbrev-ref HEAD)/mindocs/html
```

---

## Syncing Operator CRD Docs

To sync Operator CRD documentation:

```bash
make sync-operator-crd
```

This script:
- Downloads and converts `tenant_crd.adoc` from `github.com/lgcorzo/operator`.
- Downloads Operator Helm `values.yaml` and Tenant Helm `values.yaml`.
- Converts AsciiDoc to XML/Markdown and applies Sphinx ingest formatting.

---

## License

This project is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/legalcode). See [CONTRIBUTING.md](https://github.com/lgcorzo/docs/tree/master/CONTRIBUTING.md) for contribution guidelines.
