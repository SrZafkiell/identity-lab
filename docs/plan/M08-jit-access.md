# M08 - Just-in-time access

Track: C. Size: XL. Depends on: M07 (connectors), M11 (policies). Unlocks: M09, M14.

## Goal

No standing admin access: people request elevated access for a time window with a
reason, an approver approves, access is granted and then removed automatically, and
everything is auditable, even when the host is off at the moment of expiry.

## Design notes

- `governd` (Go, PostgreSQL) owns requests, approvals, grants and the audit log.
- Targets through the connectors: Keycloak groups (which drive app roles, SSH
  principals and OpenBao policies), AWS IAM Identity Center account assignments and
  GitHub team membership. Privileged access never flows through groups that provisiond
  syncs.
- Fail closed on AWS: each grant uses its own permission set with an explicit deny once
  the expiry time has passed (`aws:CurrentTime`) and a one-hour session duration. AWS
  stops the access on time even if the host is off, and the deny also cuts role
  sessions that were already issued, which removing an assignment alone does not do.
- No credential outlives its grant: SSH certificates, OpenBao tokens and AWS sessions
  issued under a grant expire no later than the grant.
- Host off: expiry runs on a durable job queue in PostgreSQL; on start, the reconcile
  loop removes every overdue grant before any request is served, then removes any
  elevated grant without an active request. GitHub has no expiring team membership, so
  GitHub grants have short maximum durations and the overrun while the host is off is
  an accepted, documented risk.
- Policy in OPA (M11): who may request what, the maximum duration, how many approvers,
  no self-approval, stricter rules for production-like targets.
- Break-glass: immediate access without approval, an alert to every approver and a
  mandatory review afterwards.
- The audit log is append-only and hash-chained, so a changed or deleted record is
  detectable.
- Channels: CLI (`idlab access request`), portal (M14), email (Mailpit in the lab) and
  chat. Chat uses Slack Socket Mode or a Discord gateway bot, both outbound connections,
  because interactive callbacks to a public URL would break the this-machine-only rule.

## Issues

- M08-01 [adr] JIT data model, fail-closed expiry and audit log -- Done when: accepted.
- M08-02 [feature] governd service skeleton -- Done when: an API with endpoints protected by Keycloak OIDC, PostgreSQL migrations, health and metrics.
- M08-03 [feature] Eligibility and approval policies in OPA -- Done when: policies decide eligibility, duration and approvers, and policy tests cover every rule.
- M08-04 [feature] Request and approval workflow -- Done when: request, approve, deny, cancel and expire run on a tested state machine with no self-approval.
- M08-05 [feature] Grant and revoke through connectors -- Done when: Keycloak group, AWS account assignment and GitHub team grants work and are revoked on expiry.
- M08-06 [security] Fail-closed AWS grants -- Done when: each AWS grant uses its own permission set with an explicit deny after expiry and a one-hour session duration, and access ends on time with the stack stopped (tested).
- M08-07 [security] Credentials never outlive the grant -- Done when: SSH certificates, OpenBao tokens and AWS sessions issued under a grant expire no later than the grant (tested per target).
- M08-08 [feature] Durable expiry and reconcile -- Done when: expiry survives restarts; on start, overdue grants are removed before any request is served; a grant left behind by a crash is found and removed (tested).
- M08-09 [feature] Hash-chained audit log -- Done when: every event is recorded and a verify command detects a modified or deleted record.
- M08-10 [feature] Break-glass -- Done when: access is immediate, approvers are alerted, and a post-incident review task is created automatically.
- M08-11 [feature] Email and chat notifications -- Done when: approvers get actionable messages; chat runs over Slack Socket Mode or a Discord gateway bot with no public callback URL, and every action is verified.
- M08-12 [feature] CLI: `idlab access request`, `list` and `approve` -- Done when: the full flow works from the terminal.
- M08-13 [test] End-to-end tests -- Done when: request, approval, grant, expiry and revocation are tested for each target type, with the revocation lag in the apps measured.
- M08-14 [test] Host-off test -- Done when: the stack is stopped across an expiry and restarted; AWS access was already denied at expiry, and other overdue grants are removed first on start (time measured).
- M08-15 [security] Abuse cases -- Done when: tests cover self-approval, approval replay, approving an expired request and privilege escalation through nested groups.
- M08-16 [docs] JIT guide -- Done when: user, approver and admin guides, the policy reference and the accepted GitHub overrun risk are written.

## Measures

- Time from approval to working access, per target.
- Time from expiry to confirmed revocation; time to remove overdue grants after the
  host starts.
- Standing privileged grants on demo data (target 0).
