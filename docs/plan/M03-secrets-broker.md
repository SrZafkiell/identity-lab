# M03 - Secrets broker

Track: A. Size: M. Depends on: M04. Unlocks: dynamic credentials for the M05 apps,
connector secrets for M07 and M08.

## Goal

No `.env` files with real secrets: people get secrets after SSO, services get
short-lived database credentials, and every read is audited.

## Design notes

- ADR: OpenBao vs HashiCorp Vault vs Infisical. Working choice: OpenBao, the open-source
  fork of Vault under MPL 2.0 hosted by the Linux Foundation, with OIDC auth, dynamic
  database credentials and audit devices.
- People: OIDC auth with Keycloak; Keycloak groups map to policies. Policies are read at
  login, so a token from a JIT-granted policy gets a TTL no longer than the grant.
- Services: AppRole or JWT auth; PostgreSQL dynamic credentials with a one-hour lease.
  The M05 apps start with static database credentials and move here when M03 lands.
- Connector credentials (the Identity Center SCIM token, the GitHub app key) move here
  from the SOPS files.
- The unseal strategy (Shamir keys kept offline vs an auto-unseal option) is decided in
  the ADR, weighing convenience against protection on a single host.

## Issues

- M03-01 [adr] Secrets broker choice and unseal strategy -- Done when: accepted.
- M03-02 [feature] OpenBao service in the `secrets` profile -- Done when: integrated storage, TLS through the proxy, and an audit device whose log is shipped to Loki.
- M03-03 [feature] OIDC login for people -- Done when: Keycloak groups map to OpenBao policies, tokens are short-lived, and a token from a JIT-granted policy never outlives the grant.
- M03-04 [feature] Dynamic PostgreSQL credentials -- Done when: demo apps get their own database users with a one-hour lease, renewed automatically and revoked on shutdown.
- M03-05 [feature] KV store for connector secrets -- Done when: provisiond and governd read their secrets from OpenBao, not from files.
- M03-06 [feature] CLI: `idlab secrets get` and `idlab secrets run -- <command>` -- Done when: secrets reach one process as environment variables and are never written to disk or shell history.
- M03-07 [security] Policy tests -- Done when: every policy is least-privilege and a test checks its allowed and denied paths.
- M03-08 [test] Integration tests -- Done when: login, lease renewal, revocation and audit entries are covered.
- M03-09 [docs] Runbook -- Done when: unseal, root rotation, revoking all leases, backup and restore are documented.

## Measures

- Long-lived secrets left in files outside bootstrap (target 0).
- Database credential lifetime.
