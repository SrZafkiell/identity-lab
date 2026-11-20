# M09 - Access reviews

Track: C. Size: L. Depends on: M08 (governd and connectors). Unlocks: review screens in
M14.

## Goal

Periodic, provable reviews of who has access to what, with revocation built in and
evidence an auditor can use.

## Design notes

- Campaigns over any connector: Keycloak groups and roles, AWS account assignments,
  GitHub organization and team membership.
- Reviewers: the manager from the HR source or the resource owner, with delegation and
  escalation when a reviewer does not answer.
- Decisions: keep, revoke (executed through the connectors) or flag, each with a reason.
- Evidence: a CSV and a signed JSON report per campaign showing who reviewed what, when,
  and what changed.
- Control mapping for SOC 2 (CC6.1 to CC6.3) and ISO 27001:2022 Annex A (5.15, 5.16,
  5.18, 8.2).

## Issues

- M09-01 [adr] Review model -- Done when: accepted, covering snapshots, reviewer assignment and evidence format.
- M09-02 [feature] Campaign definition and scheduling -- Done when: scope, reviewers, deadline and recurrence are configurable and campaigns start on schedule.
- M09-03 [feature] Entitlement snapshot -- Done when: a campaign freezes the access it reviews and shows later changes as differences.
- M09-04 [feature] Review decisions -- Done when: keep, revoke and flag with reasons and bulk actions work, and revocations run through the connectors and are confirmed.
- M09-05 [feature] Reminders, delegation and escalation -- Done when: email reminders go out and overdue items escalate to the next reviewer.
- M09-06 [feature] Evidence export -- Done when: each campaign produces a CSV and a signed JSON report, verifiable with a documented command.
- M09-07 [feature] Risk hints -- Done when: reviewers see idaudit findings (M10) and last-used data next to each entitlement.
- M09-08 [docs] Control mapping -- Done when: each feature is mapped to SOC 2 CC6 and ISO 27001:2022 controls, with the evidence it produces.
- M09-09 [test] Campaign test -- Done when: a full campaign on demo data runs in CI, revocations included.

## Measures

- Campaign size and completion time on demo data.
- Revocations executed and confirmed per campaign.
