# P2 - Secure delivery

Track: D. Size: L. Depends on: P1. Unlocks: deploys of every module.

## Goal

Every image that runs on the host was built by CI from `main`, scanned, signed and
recorded; the host runs only what it can verify.

## Design notes

- Build once: every merge to `main` builds, scans and signs the images and publishes a
  manifest of their digests. The deploy agent follows the newest `main` manifest. A
  `vX.Y.Z` tag promotes digests that were already built and verified; nothing is
  rebuilt for a release.
- cosign keyless signing with GitHub OIDC. Verification checks the build workflow on
  `refs/heads/main` and the repository ID recorded in the signing certificate, so a
  renamed or re-created repository cannot pass.
- An SBOM (Syft) and build provenance are attached to every image as attestations.
- Registry names must be lowercase; the owner name has capitals, so workflows lowercase
  it.
- Images are public: no secrets or environment configuration inside them, checked by a
  secret scan of every image.
- Base images come from registries without anonymous pull limits, or are pulled with
  authentication. Third-party images are pinned by digest, and their signatures are
  checked where the publisher signs.
- Rollback: Keycloak and other services migrate their schema on start and cannot
  downgrade. A release that migrates a database takes a backup first, and rolling back
  across a migration means restoring that backup. Our own services use
  expand-and-contract migrations, so the previous version still runs on the new schema.
- `deploy-agent` (Go) runs on the host, verifies every signature and attestation,
  deploys by digest, runs health checks, applies Terraform and rolls back on failure. It
  has no listening port. It is the only container with write access to the Docker
  engine, which is root-equivalent on the Docker VM; the ADR records this risk and the
  mitigations.

## Issues

- P2-01 [ci] Build workflow -- Done when: service images build on pull requests (no push) and on `main` (push to GHCR with lowercase names), with caching.
- P2-02 [security] Image scanning gate -- Done when: Trivy fails the build on fixable critical or high CVEs and on secrets found in an image, and uploads results to code scanning.
- P2-03 [security] Signing and attestations -- Done when: images are signed with cosign keyless, SBOM and provenance attestations are attached, and verification checks the workflow identity and the repository ID.
- P2-04 [ci] Release workflow -- Done when: a `vX.Y.Z` tag promotes the digests built on `main` into a release manifest with release notes from Conventional Commits and attaches the M01 binaries, without rebuilding anything.
- P2-05 [security] Upstream images -- Done when: every third-party image is pinned by digest, pulled from a registry without anonymous limits or with authentication, and signature-checked where the publisher signs.
- P2-06 [adr] Deploy agent and its access to Docker -- Done when: accepted, with the risks and mitigations.
- P2-07 [feature] Deploy agent: poll and verify -- Done when: the agent detects a new `main` manifest, verifies signatures and attestations, and refuses unsigned or wrongly signed images.
- P2-08 [feature] Deploy agent: deploy, health check, rollback -- Done when: it deploys by digest, waits for health checks and rolls back on failure; a release that migrates a database takes a backup first and is rolled back by restore, never by an image downgrade.
- P2-09 [feature] Deploy agent: Terraform apply -- Done when: IdP configuration is applied after a successful deploy, with the plan kept in the deploy log.
- P2-10 [test] Agent failure tests -- Done when: tests cover a tampered image, a missing signature, a wrong signer, a failing health check, an interrupted deploy and a failed migrating release.
- P2-11 [test] Integration job -- Done when: CI brings up `core` and the changed profiles on the runner and runs smoke tests.
- P2-12 [security] DAST baseline -- Done when: a ZAP baseline scan runs against the running stack in CI and findings are triaged.
- P2-13 [docs] Delivery runbook -- Done when: release, deploy, rollback, restore after a failed migration and signer identity checks are documented.

## Measures

- Time from merge to running on the host.
- Rollback time.
- Critical and high CVEs in released images (target 0).
