# M12 - AWS federation

Track: B. Size: M. Depends on: M04 and the sandbox organization from P0. Unlocks: AWS
targets in M07 and M08, sandbox login in M01.

## Goal

People in the lab sign in to AWS with their lab identity: Keycloak is the identity
provider for IAM Identity Center, users and groups are provisioned to it, and there are
no IAM users besides break-glass.

## Design notes

- The sandbox is a dedicated AWS Organization used only for the lab (set up in P0-23),
  managed in Terraform with state on the host.
- SAML 2.0 between Keycloak and IAM Identity Center works with an IdP that only this
  machine can reach, because the browser carries the assertion; AWS never calls the IdP.
  A spike confirms that AWS accepts the lab's SAML metadata and `.internal` URLs before
  anything else is built.
- OIDC web identity federation (AssumeRoleWithWebIdentity) is not used: STS must fetch
  the issuer's keys from a public URL, which breaks the this-machine-only rule. The ADR
  records this and the alternative of publishing only the discovery document and keys.
- With Keycloak as the identity source, nobody can sign in to Identity Center while the
  host is off. That is why the organization is dedicated to the lab and why break-glass
  access is built and tested.
- Users and groups reach Identity Center through `provisiond` (M07, SCIM).
- Permission sets and account assignments for normal access live in Terraform; elevated
  access comes only through JIT (M08), with per-grant permission sets.
- Stretch: direct IAM SAML role federation for the CLI (AssumeRoleWithSAML), designed in
  a spike first.

## Issues

- M12-01 [spike] Identity Center with a workstation-only IdP -- Done when: AWS accepts the lab's SAML metadata and URLs and a test user signs in, before the rest of M12 starts.
- M12-02 [adr] AWS federation design -- Done when: accepted, with the SAML choice and the reason OIDC web identity is not used.
- M12-03 [feature] Terraform for the sandbox -- Done when: permission sets, group assignments and the external identity provider settings are declared in Terraform.
- M12-04 [feature] Keycloak SAML client for Identity Center -- Done when: login from the AWS access portal goes through Keycloak with MFA, and the attribute mapping is documented.
- M12-05 [feature] SCIM to Identity Center -- Done when: provisiond pushes users and groups, its token lives in OpenBao (M03), and the token's expiry is in the expiry inventory (M13).
- M12-06 [security] Break-glass access -- Done when: emergency access to the management account works without Keycloak, is documented and tested, and any use raises an alert.
- M12-07 [feature] CLI integration -- Done when: idlab (M01) discovers the sandbox accounts through the Keycloak-backed SSO and opens the console.
- M12-08 [spike] CLI SAML role federation -- Done when: feasibility and a design are documented (stretch).
- M12-09 [test] Nightly federation test -- Done when: a Playwright run logs in through Keycloak and checks the expected accounts and roles.
- M12-10 [docs] Federation guide -- Done when: setup, attribute mapping and recovery from an IdP outage are documented.

## Measures

- IAM users in the sandbox (target 0 besides break-glass).
- Time from a Keycloak group change to working AWS access.
