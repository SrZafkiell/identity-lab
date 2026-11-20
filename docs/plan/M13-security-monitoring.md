# M13 - Security monitoring

Track: D. Size: L. Depends on: P1 and M04; more sources join as their modules land.

## Goal

See identity attacks and risky changes as they happen, with detections written as code
and tested like code.

## Design notes

- Sources: Keycloak user and admin events, the OpenBao audit log, step-ca issuance, the
  governd audit log, OPA decision logs and CloudTrail management events (through the
  free event history API, which is per region and rate-limited, so every region in use
  is polled).
- Grafana Alloy collects, Loki stores, and Grafana alerting evaluates the rules and
  routes alerts to email and chat.
- Events are parsed into one common schema with OCSF-style fields.
- Every detection has an id, a MITRE ATT&CK mapping, a severity, a runbook link, and
  test fixtures: attack events that must fire it and normal traffic that must not.
- Some detections (impossible travel) need data a single host cannot produce; they are
  tested with synthetic events and labeled as such.
- The attack simulator only targets lab hostnames and is rate-limited, so it cannot be
  pointed at a real system.
- Every credential and certificate with an expiry date is tracked, so nothing expires
  unnoticed.

## Detections (first set)

- Password spraying and brute force against Keycloak.
- MFA removed or reset for a user; an admin role granted outside JIT.
- Break-glass used; a JIT grant without a matching approval; overdue grants found at
  start (the host was off at expiry).
- Login from a new country or impossible travel (synthetic data).
- AWS root user sign-in; console sign-in without MFA; an IAM user or access key created.
- Mass deprovisioning or mass SSH certificate issuance.
- OpenBao root token use or policy change.
- A broken hash chain in the JIT audit log.
- Clock skew above the threshold on any container.

## Issues

- M13-01 [adr] Detections as code and their tests -- Done when: accepted.
- M13-02 [feature] Event pipelines per source -- Done when: each source is parsed into the common schema and the parsing is covered by tests.
- M13-03 [feature] CloudTrail poller -- Done when: it polls the event history API for every region in use within its rate limits, with no duplicates and a cursor kept across restarts.
- M13-04 [feature] Detection rules -- Done when: the first set above exists as Grafana alert rules provisioned from code.
- M13-05 [test] Detection tests in CI -- Done when: fixtures are replayed into a test Loki, and each rule fires on its attack fixture and stays quiet on normal traffic.
- M13-06 [feature] Alert routing -- Done when: alerts route to email and chat by severity, with deduplication and silences in code.
- M13-07 [feature] Dashboards -- Done when: identity overview, JIT activity, provisioning and audit-chain health dashboards are provisioned from code.
- M13-08 [feature] Expiry inventory -- Done when: every credential and certificate with an expiry (SCIM token, app keys, CA and leaf certificates, client secrets) is listed, with alerts before it expires.
- M13-09 [feature] Attack simulator -- Done when: one command runs spraying, MFA reset and impossible-travel scenarios against the lab, refuses any target outside the lab domain, and is rate-limited.
- M13-10 [docs] Detection catalog and runbooks -- Done when: every detection is documented with its ATT&CK mapping and a response runbook.

## Measures

- Detection count and the share covered by tests.
- Time from an attack event to the alert on the host.
