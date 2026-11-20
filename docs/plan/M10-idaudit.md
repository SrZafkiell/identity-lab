# M10 - Access audit (idaudit)

Track: C. Size: L. Depends on: P0 (live Keycloak rules need M04). Unlocks: risk hints in
M09 and CI gates on IdP changes.

## Goal

A Go CLI that finds risky identity and access configuration across Keycloak, AWS and
GitHub, explains each finding, and can block a pull request.

## Design notes

- Sources: live APIs with read-only credentials, or exported files (realm JSON, the IAM
  credential report, saved API responses), so CI and tests run without any access.
- Every rule has an id, a severity, a description, a remediation and references (the CIS
  AWS Foundations Benchmark where it applies).
- Output: table, JSON, Markdown and SARIF (shown in GitHub code scanning).
- A suppressions file needs a reason and an expiry date per entry; expired suppressions
  fail the run.
- The exit code follows a severity threshold, so it can gate CI.
- The GitHub test organization is on the Free plan; rules that need paid features
  report "not applicable" instead of failing.
- CI reaches the sandbox through GitHub OIDC. Jobs that use an environment get an
  environment-form `sub` claim, and repositories created after 2026-07-15 carry
  immutable owner and repository IDs in it, so the AWS trust policy is written from a
  `sub` value observed in a real run.

## Rules (first set)

- Keycloak: implicit flow or direct access grants enabled; wildcard redirect URIs;
  public clients without PKCE; long token lifetimes; brute-force detection off; users
  with admin roles; service accounts with realm-admin; users without MFA; unused
  accounts; oversized composite roles.
- AWS: root access keys or no root MFA; IAM users without MFA; access keys unused or
  older than 90 days; AdministratorAccess on users or broad groups; role trust policies
  open to any principal or to other accounts without conditions; admin permission sets
  assigned widely; a password policy weaker than the baseline.
- GitHub: 2FA not required; outside collaborators with admin; default workflow token
  permission set to write; Actions allowed from any source; default branches without
  protection; deploy keys with write access.

## Issues

- M10-01 [adr] idaudit design: sources, rule model, output formats -- Done when: accepted.
- M10-02 [feature] Core: rule engine, finding model and outputs -- Done when: table, JSON, Markdown and SARIF outputs exist and have golden-file tests.
- M10-03 [feature] Suppressions with expiry -- Done when: every suppression needs a reason and an expiry, and expired ones fail the run.
- M10-04 [feature] Keycloak source and rules -- Done when: the rules work on exported realms and the live API, with a positive and a negative fixture per rule.
- M10-05 [feature] AWS source and rules -- Done when: the rules work on saved API responses and on the live sandbox through a read-only role.
- M10-06 [feature] GitHub source and rules -- Done when: the rules work on saved API responses and on the test organization with read-only credentials, and rules that need paid features report not applicable.
- M10-07 [security] GitHub OIDC trust for the sandbox -- Done when: the AWS role used by CI trusts only this repository's jobs, the condition is written from an observed `sub` value, and no static AWS keys exist in GitHub.
- M10-08 [ci] Gate IdP changes -- Done when: CI runs idaudit against the ephemeral Keycloak after Terraform apply, fails on high findings and uploads SARIF.
- M10-09 [ci] Scheduled audits -- Done when: a nightly run on `main` audits the sandbox and the test organization and publishes results to code scanning.
- M10-10 [ci] Release binaries -- Done when: signed Windows, macOS and Linux binaries ship with an SBOM.
- M10-11 [docs] Rule reference -- Done when: a generated page per rule shows its rationale and remediation.

## Measures

- Rules per source.
- Findings on the default vs the hardened configuration.
