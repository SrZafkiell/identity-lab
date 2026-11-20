# M14 - Portal

Track: C. Size: L. Depends on: M08 and M09 (some views also use M01 and M10 data).

## Goal

One web UI for people and approvers: see your access, request more, approve requests,
run reviews and see audit findings.

## Design notes

- React and TypeScript front end. A Go backend-for-frontend handles the OIDC login,
  keeps tokens on the server and gives the browser only an httpOnly, SameSite session
  cookie.
- Strict Content Security Policy, CSRF protection on every state-changing call, no
  third-party scripts.
- Accessibility target: WCAG 2.2 AA, checked with axe in the browser tests.
- Views: my access, request access, approvals inbox, review campaigns, audit findings,
  AWS accounts with console links, and connector health for admins.

## Issues

- M14-01 [adr] Portal architecture: SPA with a backend-for-frontend -- Done when: accepted.
- M14-02 [feature] BFF: OIDC login, session, CSRF and API proxy -- Done when: tokens never reach the browser (tested) and the session ends on Keycloak logout.
- M14-03 [feature] App shell, navigation and components -- Done when: layout, light and dark themes, keyboard navigation and accessible components are in place.
- M14-04 [feature] My access and request access -- Done when: current grants show with their expiry and the request form follows OPA eligibility.
- M14-05 [feature] Approvals inbox -- Done when: approvers approve or deny with a reason, behind a step-up (M06).
- M14-06 [feature] Review campaigns -- Done when: a reviewer completes a campaign end to end in the UI.
- M14-07 [feature] Audit findings view -- Done when: idaudit results have filters and links to the remediation.
- M14-08 [feature] AWS accounts view -- Done when: the accounts and roles a user can reach are listed with console links.
- M14-09 [feature] Admin: connector health -- Done when: status, last reconcile and errors are shown per connector.
- M14-10 [test] Browser tests and accessibility checks -- Done when: Playwright covers each view and axe reports no serious or critical issues.
- M14-11 [security] Portal security tests -- Done when: the CSP is verified, CSRF is tested, and the ZAP baseline has no high findings.

## Measures

- Lighthouse performance and accessibility scores.
- Playwright test count.
