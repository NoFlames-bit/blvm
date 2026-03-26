# SDLC (Software Development Lifecycle) — Bitcoin Commons Ecosystem

This document summarizes the project’s SDLC as implemented across the BTCDecoded repositories: how changes are proposed, reviewed/enforced, validated in CI, built deterministically, and released/deployed.

It is derived from the repository documentation and build/release automation in this workspace.

## 1. Propose / Classify the Change

### 1.1 Code change happens in the appropriate repo
- Normal development is done by opening PRs against the relevant component repository (e.g., `blvm-consensus`, `blvm-protocol`, `blvm-node`, `blvm`, `blvm-sdk`, `blvm-commons`, `governance-app`).

### 1.2 Governance tier determines how “hard” the change must be reviewed
- The governance system uses a tier model (time window + maintainer signature thresholds) and separates:
  - repository access governance (binding)
  - protocol guidance (advisory)
- Tier and signature timing are enforced as part of the PR lifecycle.

Evidence:
- `BTCDecoded/governance/README.md`

## 2. Implement (Repo-Specific Engineering Constraints)

### 2.1 Consensus-critical changes have stricter correctness requirements
- Consensus code must match Bitcoin Core behavior and Orange Paper specifications, with high test coverage expectations and possible formal verification/spec-lock annotations.

Evidence:
- `BTCDecoded/blvm-consensus/CONTRIBUTING.md`

### 2.2 Node-critical changes require compatibility, performance, and safety considerations
- Node changes must preserve Bitcoin network compatibility and avoid consensus rule modifications (consensus belongs in `blvm-consensus`).

Evidence:
- `BTCDecoded/blvm-node/CONTRIBUTING.md`

### 2.3 Build/release system changes must preserve cross-repo compatibility
- Changes to build/release orchestration and version coordination are treated as ecosystem-wide critical paths.

Evidence:
- `BTCDecoded/blvm/CONTRIBUTING.md`

## 3. Review, Sign, and (If Needed) Veto (Merge Gate)

### 3.1 PR lifecycle is tier-driven
Based on the governance tier classification, the PR is gated by:
1. automatic tier classification (with possible manual override)
2. required maintainer signatures
3. required review period duration per tier
4. (tier 3+) economic node veto window, if the change crosses consensus-adjacent thresholds
5. merge enabled only when requirements are met

Evidence (mechanics described):
- `BTCDecoded/governance/README.md`

## 4. Validate in CI (Quality + Security Gates)

### 4.1 Rust ecosystem CI checks
- CI builds and validates the codebase on self-hosted runners.
- Common gates include:
  - formatting checks (`cargo fmt -- --check`)
  - linting (`cargo clippy -- -D warnings`)
  - security scanning (`cargo audit`)
  - compilation/tests (`cargo check`, `cargo test`)

Evidence:
- `BTCDecoded/blvm/.github/workflows/ci.yml`

## 5. Deterministic Build Verification (Reproducibility)

### 5.1 Deterministic build wrapper produces hashes
- Builds use locked dependencies (`--locked`) and deterministic build flags where available.
- Outputs are hashed into `SHA256SUMS` (including `Cargo.lock`) to support later reproducibility checks.

Evidence:
- `BTCDecoded/blvm/tools/det_build.sh`

## 6. Release Engineering (Multi-Repo Orchestration)

### 6.1 “Release set” defined via a single source of truth
- Release versions for each component are pinned via `versions.toml`.
- Release sets are then built in dependency/topology order.

Evidence:
- `BTCDecoded/blvm/docs/RELEASE_PROCESS.md`
- `BTCDecoded/blvm/docs/workflows/WORKFLOW_METHODOLOGY.md`
- `BTCDecoded/blvm/RELEASE_SET.md`

### 6.2 Local release set build (clone tags → deterministic builds → hashes)
- A local orchestrator script can check out the exact tags defined by `versions.toml`.
- It performs deterministic builds and aggregates hashes into optional manifests.

Evidence:
- `BTCDecoded/blvm/tools/build_release_set.sh`

### 6.3 Official release pipeline design (base vs experimental + QA)
- Official releases are designed to include:
  - base and experimental variants
  - deterministic verification (hash compare after clean rebuild)
  - full test execution (with stated doctest exclusion for Phase 1 speed)
  - tagging across component repos and creation of a GitHub release with artifacts + checksums + release notes

Evidence:
- `BTCDecoded/blvm/docs/RELEASE_PROCESS.md`
- `BTCDecoded/blvm/docs/OFFICIAL_RELEASE_PIPELINE.md`

## 7. Deployment / Activation Signaling

### 7.1 Governance + orchestration alignment
- The workflow methodology describes how the commons/orchestrator signals a deployment action to `governance-app`.
- Governance activation is staged: Phase 1 infrastructure exists, later phases activate enforcement.

Evidence:
- `BTCDecoded/blvm/docs/workflows/WORKFLOW_METHODOLOGY.md`
- `BTCDecoded/governance/README.md`

## 8. Security Response (Responsible Disclosure)

### 8.1 Security reporting and response timelines
- The orchestration/release automation repositories include security policies emphasizing responsible disclosure (no public issue) and defined response timelines.

Evidence:
- `BTCDecoded/blvm/SECURITY.md`
- `BTCDecoded/governance/SECURITY.md`

## Evidence Links (Key Files)

- Governance + PR enforcement model: `BTCDecoded/governance/README.md`
- Governance change process: `BTCDecoded/governance/CONTRIBUTING.md`
- Repo contribution constraints:
  - `BTCDecoded/blvm-consensus/CONTRIBUTING.md`
  - `BTCDecoded/blvm-node/CONTRIBUTING.md`
  - `BTCDecoded/blvm/CONTRIBUTING.md`
- CI gates: `BTCDecoded/blvm/.github/workflows/ci.yml`
- Deterministic build wrapper: `BTCDecoded/blvm/tools/det_build.sh`
- Release orchestration (release set): `BTCDecoded/blvm/tools/build_release_set.sh`
- Release set coordination: `BTCDecoded/blvm/RELEASE_SET.md`
- Release process & official pipeline:
  - `BTCDecoded/blvm/docs/RELEASE_PROCESS.md`
  - `BTCDecoded/blvm/docs/OFFICIAL_RELEASE_PIPELINE.md`
- Org workflow model: `BTCDecoded/blvm/docs/workflows/WORKFLOW_METHODOLOGY.md`
- Build system/orchestration docs: `BTCDecoded/blvm/docs/build/BUILD_SYSTEM.md`

