# M04 - Identity provider

Track: B. Size: M. Depends on: P1. Unlocks: M02, M03, M05, M06, M07, M10 (live rules),
M12, M13.

## Goal

A hardened Keycloak whose whole configuration lives in Terraform and is reviewed in pull
requests.

## Design notes

- Keycloak 26.8 or later in production mode (optimized build), behind the proxy, with
  health and metrics endpoints.
- Two realms: `master` for administration only and `lab` for users and apps. The admin
  console has its own hostname, so it can be restricted separately.
- Terraform with the official provider (`keycloak/keycloak`). CI posts the plan on the
  pull request, run against an ephemeral Keycloak.
- State: Terraform keeps secrets in state in plain text. Secrets stay out of state with
  ephemeral resources and write-only arguments (Terraform 1.10 and later) wherever the
  providers support them; whatever remains is protected by disk encryption and
  encrypted backups, and state never goes to git. OpenTofu stays a drop-in fallback if
  this proves weak.
- Ownership: Terraform owns structure (realms, clients, flows, roles, role mappings and
  the definitions of privileged groups). It never declares group memberships, because
  SCIM (M07) and JIT (M08) write those.
- The provider can lag new Keycloak features (for example the SCIM settings); gaps are
  applied by scripted, idempotent steps and tracked.
- Realm baseline: password policy, brute-force detection, short access tokens with
  refresh token rotation, user and admin events recorded.
- Demo users use `example.com` addresses, and all mail goes to Mailpit.

## Issues

- M04-01 [adr] Identity provider choice -- Done when: Keycloak is accepted against Authentik, ZITADEL and Authelia, with reasons.
- M04-02 [adr] Terraform state and secrets -- Done when: accepted, covering the backend and where state lives, secrets kept out of state with ephemeral resources and write-only arguments where supported, protection of what remains, and never in git.
- M04-03 [feature] Keycloak service in the `core` profile -- Done when: production mode, PostgreSQL backend, health checks, metrics scraped and a separate admin hostname.
- M04-04 [feature] Terraform root module for the lab realm -- Done when: realm settings, password policy, brute-force detection, token lifetimes, events and SMTP to Mailpit are declared.
- M04-05 [feature] Groups and roles model -- Done when: a documented model (departments, teams, app roles, privileged roles) is implemented in Terraform, following the ownership rules in PLAN.md.
- M04-06 [security] Admin hardening -- Done when: the master realm holds only break-glass admins with MFA, and Terraform uses a service account with only the roles it needs.
- M04-07 [ci] Terraform checks -- Done when: fmt, validate, tflint and Trivy config run on pull requests, and the plan against an ephemeral Keycloak is posted as a comment.
- M04-08 [feature] Drift detection -- Done when: a nightly plan on the host must come back empty, and any drift raises an alert.
- M04-09 [chore] Provider gaps -- Done when: Keycloak settings the provider cannot manage yet are applied by a scripted, idempotent step, and each gap is tracked as an issue.
- M04-10 [feature] Seed data -- Done when: a generated set of demo users and groups (no real people, `example.com` addresses) loads with one command.
- M04-11 [test] Load test -- Done when: a k6 script measures login throughput and latency on the host, and the results are recorded as single-host numbers.
- M04-12 [docs] Identity model and runbook -- Done when: realms, groups, roles, ownership, upgrade steps and admin recovery are documented.

## Measures

- Login latency p50 and p95, and logins per second (k6, single host).
