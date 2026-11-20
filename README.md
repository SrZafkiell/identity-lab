# identity-lab

A self-hosted identity and access platform, built end to end as a reference: single
sign-on for a few sample apps, automated user provisioning, enforced MFA, and a small
tool that audits access policies for more permission than they need.

Everything runs on one local machine with Docker. No cloud account is needed to run it.

> Status: planning. The plan, the issues and the milestones live in this repository.

## Goals

- **Single sign-on** with OpenID Connect and SAML 2.0 for at least two sample
  applications (one modern web app, one legacy-style app).
- **Identity provider** self-hosted (Keycloak is the working choice; the decision gets
  an ADR).
- **Provisioning**: users and groups created, changed and removed through SCIM 2.0, so
  access follows the source of truth instead of manual edits.
- **MFA** enforced (TOTP and WebAuthn), with step-up authentication for an admin-only
  action.
- **Role-based access** mapped from groups to application roles, and a joiner / mover /
  leaver flow that can be demonstrated in a few minutes.
- **Access audit tool**: reads the realm and application configuration and reports
  over-privileged roles, unused accounts, stale sessions and risky client settings.
- **Infrastructure as code**: the identity provider, realms, clients and roles are
  declared in code (Terraform), never clicked into place.

## Engineering standards

- Work tracked as GitHub issues and milestones; every change goes through a pull request.
- CI on every pull request: lint, unit and integration tests, secret scanning,
  dependency and container image scanning, and static analysis.
- CD to the local machine, only from `main`, with signed and scanned images.
- Architecture Decision Records in `docs/adr/` for every decision that matters.
- Threat model for the platform and the CI/CD path.
- Measured results published in this README (test counts, login latency, time to
  provision and deprovision a user).
- A short walkthrough video once the first milestone ships.

## Out of scope (for now)

- Managed cloud identity services (AWS IAM Identity Center, Entra ID, Okta).
- High availability of the identity provider across machines.

## License

To be decided before the first release.
