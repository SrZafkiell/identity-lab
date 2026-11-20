# M07 - Provisioning (SCIM)

Track: B. Size: L. Depends on: M04 (the AWS target also needs M12). Unlocks: M08, M09.

## Goal

Access follows a source of truth: joiners get access on day one, movers lose what they
no longer need, leavers lose everything everywhere, and every step is logged.

## Design notes

- Inbound: Keycloak's native SCIM API (supported since Keycloak 26.8, users and groups)
  receives identities from a simulated HR system.
- Outbound: Keycloak does not push to downstream systems, so `provisiond` (Go) does. It
  reconciles Keycloak, the source for downstream systems, against each target on a
  schedule and on change events.
- Targets: AWS IAM Identity Center (SCIM), the OIDC demo app (a small SCIM server added
  to it) and a GitHub test organization (REST API, because GitHub's SCIM needs an
  Enterprise plan).
- Ownership: provisiond writes only users and non-privileged groups downstream.
  Privileged groups are never synced; privileged access comes only from JIT (M08), so it
  cannot outlive a grant through a synced group.
- Connectors differ (Identity Center has no nested groups and limited SCIM filters), so
  the shared connector interface in `internal/connectors` (list identities, list
  entitlements, grant, revoke) carries capability flags; Keycloak subgroups are
  flattened for targets without nesting. M08, M09 and M10 reuse the interface.
- Synthetic demo users cannot have GitHub accounts (one free account per person), so the
  GitHub connector is tested against a mocked API in CI and, nightly, with the owner's
  account and one machine account.
- Safety rails: provisiond never removes organization owners or the last admin of any
  target.
- Reconciliation is idempotent and reports drift; target deletions are soft by default
  (deactivate first, delete after a grace period).

## Issues

- M07-01 [spike] Keycloak native SCIM API on the pinned version -- Done when: coverage of users, groups, PATCH, filters, auth and the per-realm switch is documented, gaps included.
- M07-02 [adr] Provisioning: native SCIM inbound, own provisioner outbound -- Done when: accepted.
- M07-03 [feature] Simulated HR source -- Done when: it generates joiner, mover and leaver events for a fixture company and sends them to the Keycloak SCIM API.
- M07-04 [feature] Connector interface and contract tests -- Done when: the interface carries capability flags (nested groups, listing, soft delete) and a contract test suite runs per capability.
- M07-05 [feature] provisiond reconcile loop -- Done when: it computes the diff, applies it idempotently, records every change and reports drift, on a schedule and on Keycloak admin events; privileged groups are never synced, and safety rails block removing owners or the last admin.
- M07-06 [feature] SCIM server in the OIDC demo app -- Done when: Users and Groups endpoints support filters, PATCH and pagination, and the SCIM filter parser is fuzz-tested.
- M07-07 [feature] Connector: OIDC demo app -- Done when: it passes the contract tests.
- M07-08 [feature] Connector: GitHub organization -- Done when: membership and team sync pass the contract tests against a mocked API in CI, and a nightly run against the test organization uses the owner's account and one machine account.
- M07-09 [feature] Connector: AWS IAM Identity Center -- Done when: users and flattened groups reach the sandbox Identity Center and the connector passes the contract tests for its capabilities.
- M07-10 [feature] Leaver flow -- Done when: a leaver event disables the Keycloak user, ends sessions, revokes issued credentials (SSH certificate serials, OpenBao leases), deprovisions every target, and one report shows the whole trail; AWS role sessions end within the permission set session duration.
- M07-11 [test] Joiner, mover and leaver end-to-end test -- Done when: it runs in CI with the app connector and a mocked GitHub, and nightly against the real test organization.
- M07-12 [docs] Provisioning guide -- Done when: the data flow, attribute mapping per target, ownership and failure handling are documented.

## Measures

- Time to provision a joiner to every target; time to fully deprovision a leaver.
- Drift found per reconcile run on demo data.
