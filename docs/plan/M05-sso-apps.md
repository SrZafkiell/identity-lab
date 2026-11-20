# M05 - SSO apps

Track: B. Size: M. Depends on: M04. Unlocks: M06, M11.

## Goal

Two working apps that show the two main enterprise SSO protocols done correctly.

## Design notes

- `demo-oidc-fastapi`: OIDC authorization code with PKCE (S256), a server-side session,
  ID token checks (issuer, audience, nonce, expiry), roles from token claims, refresh
  token rotation, RP-initiated and back-channel logout.
- `demo-saml-spring`: a Spring Security SAML 2.0 service provider that requires signed
  assertions, with optional encrypted assertions, attribute and role mapping, and single
  logout. OpenSAML is not on Maven Central; it comes from the Shibboleth repository,
  pinned and checked with Gradle dependency verification.
- Removed roles disappear from an app when its access token expires, so tokens stay
  short and back-channel logout ends sessions at once; the lag is measured.
- Both apps stay small on purpose (a simple tickets screen with user, editor and admin
  views), so the identity work is the focus.
- Stretch: DPoP sender-constrained tokens in the OIDC app.

## Issues

- M05-01 [feature] OIDC demo app (FastAPI) -- Done when: login, logout and role-based views work, and every token check has a test, rejected tokens included.
- M05-02 [feature] SAML demo app (Spring Boot) -- Done when: login, single logout and role mapping work, unsigned or tampered assertions are rejected (tested), and OpenSAML is pinned with dependency verification.
- M05-03 [feature] Clients in Terraform -- Done when: both clients, exact redirect URIs (no wildcards), mappers and role mappings are declared in Terraform.
- M05-04 [feature] Back-channel logout -- Done when: ending the session in Keycloak ends it in both apps, with the propagation time measured.
- M05-05 [test] Browser tests (Playwright) -- Done when: login, role checks and logout run in CI for both apps.
- M05-06 [security] Protocol negative tests -- Done when: tests cover replayed state, a wrong audience, expired tokens, open redirect attempts and XML signature wrapping on SAML.
- M05-07 [docs] Integration guide -- Done when: each integration is documented with sequence diagrams.

## Measures

- Login time per app and logout propagation time.
- Test count per app.
