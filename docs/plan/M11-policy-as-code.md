# M11 - Policy as code

Track: C. Size: M. Depends on: M05. Unlocks: M08.

## Goal

Authorization decisions and configuration rules written once as code, tested, versioned
and reused across the apps, JIT and CI.

## Design notes

- ADR: OPA (Rego) vs Cedar. Working choice: OPA, because the same language also checks
  Terraform plans and Compose files in CI (conftest) and drives JIT eligibility (M08).
- The apps ask OPA for a decision (user, action, resource) instead of hard-coding role
  checks; OPA runs as a sidecar and loads policy bundles that CI builds and signs.
- Decision logs go to Loki with sensitive fields masked, so every allow and deny can be
  traced.

## Issues

- M11-01 [adr] Policy engine choice -- Done when: accepted.
- M11-02 [feature] Policy layout and tests -- Done when: `policies/` has `opa test` with coverage in CI and Regal for linting.
- M11-03 [feature] App authorization with OPA -- Done when: both demo apps call OPA for every protected action, and role and attribute rules (owner, department) are tested.
- M11-04 [feature] Signed policy bundles -- Done when: CI builds and signs bundles and OPA refuses unsigned ones (tested).
- M11-05 [feature] Decision logs -- Done when: decisions reach Loki with sensitive fields masked.
- M11-06 [ci] Configuration policies with conftest -- Done when: Terraform plans and Compose files are checked in CI, extending the P1 checks.
- M11-07 [feature] JIT policies -- Done when: eligibility, duration and approver rules used by governd (M08) exist with tests.
- M11-08 [docs] Policy guide -- Done when: writing, testing and releasing a policy are documented.

## Measures

- Policy test count and coverage.
- Decision latency p95 as seen by the apps.
