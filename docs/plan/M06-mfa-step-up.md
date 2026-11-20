# M06 - MFA and step-up

Track: B. Size: M. Depends on: M05.

## Goal

Strong authentication that is required, recoverable, and stronger where the action is
riskier.

## Design notes

- TOTP and WebAuthn (passkeys and security keys); MFA required in the `lab` realm.
- Step-up: the OIDC app asks for a higher level of authentication (acr values mapped to
  levels in Keycloak) before an admin-only action, with a short maximum age.
- Recovery: recovery codes or an admin-assisted reset with an audit trail, chosen in an
  ADR based on what the pinned Keycloak version supports.
- Lockout settings balance brute-force protection against lockout used as denial of
  service.
- TOTP fails when the clock drifts; the skew check from P1 is a prerequisite.

## Issues

- M06-01 [feature] MFA enrollment and enforcement -- Done when: TOTP and WebAuthn enrollment work, MFA is required, and the flows are declared in Terraform.
- M06-02 [feature] Step-up for admin actions -- Done when: the admin action demands WebAuthn when the session is below the required level or older than the maximum age.
- M06-03 [adr] Account recovery -- Done when: accepted.
- M06-04 [feature] Account recovery -- Done when: the chosen recovery path works and every use is audited.
- M06-05 [security] Brute-force and lockout tests -- Done when: automated tests show lockout and unlock, and error messages do not reveal whether a user exists.
- M06-06 [test] Browser tests with a virtual authenticator -- Done when: Playwright covers TOTP, WebAuthn and step-up.
- M06-07 [docs] Authentication policy -- Done when: the levels of assurance, which action needs which level, and the recovery process are documented.

## Measures

- Share of demo users with MFA (target 100%).
- Time added by a step-up.
