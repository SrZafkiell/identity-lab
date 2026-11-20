# identity-lab

A self-hosted identity and access platform, built from modules: a workstation CLI that
manages AWS and SSH access, an identity core with single sign-on, MFA and provisioning,
governance with just-in-time access, access reviews and audit, and the operations around
it (signed delivery, security monitoring, a web portal).

Everything runs on one machine with Docker. Nothing is hosted in a cloud; the AWS and
GitHub modules manage access to a sandbox account and a test organization.

> Status: planning. See [the plan](docs/PLAN.md); work is tracked in issues and
> milestones.

## Modules

| Area | Module | What it does |
|---|---|---|
| Workstation | M01 AWS access manager | Profiles, saved SSO sessions, background refresh, console sign-in, credentials without plain-text files |
| Workstation | M02 SSH certificate authority | SSO login gives short-lived SSH certificates; no static keys on servers |
| Workstation | M03 Secrets broker | Secrets after SSO, dynamic database credentials, audited reads |
| Identity core | M04 Identity provider | Hardened Keycloak configured entirely in Terraform |
| Identity core | M05 SSO apps | OIDC (FastAPI) and SAML (Spring Boot) integrations done correctly |
| Identity core | M06 MFA and step-up | TOTP, WebAuthn, stronger checks for risky actions |
| Identity core | M07 Provisioning | SCIM joiner, mover and leaver flows to apps, AWS and GitHub |
| Identity core | M12 AWS federation | Keycloak as the identity provider for AWS IAM Identity Center |
| Governance | M08 Just-in-time access | Time-boxed elevated access with approvals and automatic removal |
| Governance | M09 Access reviews | Review campaigns with revocation and audit evidence |
| Governance | M10 idaudit | Finds risky identity configuration in Keycloak, AWS and GitHub |
| Governance | M11 Policy as code | OPA policies for apps, JIT and CI checks |
| Operations | M13 Security monitoring | Identity attack detections written and tested as code |
| Operations | M14 Portal | Web UI for access, approvals, reviews and findings |

Platform work: P0 foundation, P1 runtime, P2 secure delivery (signed images, pull-based
deploys) and R1, the v1.0.0 release.

## Engineering standards

- Issues, milestones and a public project board; every change through a pull request.
- CI security gates: secret scanning, CodeQL, dependency review, image and IaC scanning,
  DAST, OpenSSF Scorecard.
- Images signed with cosign, with SBOM and build provenance; the host deploys only what
  it can verify.
- ADRs for every decision that matters, a threat model and a
  [risk register](docs/RISKS.md) kept current, runbooks for operations.
- Measured results published here, each with the script or test that produced it.

## Tech

Go, Java (Spring Boot), Python (FastAPI), TypeScript (React), Keycloak, PostgreSQL,
Terraform, OPA, OpenBao, step-ca, Traefik, Grafana, Loki, Prometheus, GitHub Actions,
Docker.

## License

To be decided before the first release.
