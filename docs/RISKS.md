# Risks and constraints

Known constraints of building and running this platform on one workstation, and how the
plan handles each one. The list is reviewed at the end of every milestone; the issue IDs
point to the plan files in [plan/](plan/).

## Host

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| The host is not always on; Docker Desktop may not start at sign-in | Expiry, deprovisioning, polling and alerts stop while it is off | Grants fail closed on the target side (AWS: explicit deny after expiry); on start, overdue grants are removed first; the CloudTrail poller resumes from its cursor | M08-06, M08-08, M08-14, M13-03 |
| The WSL2 VM clock can drift after sleep | TOTP, OIDC and SAML validity windows, AWS request signing and signature checks fail | Clock-skew health check and alert; documented fix | P1-12, M13-04 |
| Docker publishes ports on every interface by default | Admin consoles reachable from the LAN | Only the proxy publishes ports, bound to 127.0.0.1, enforced by a policy check | P1-06, P1-09 |
| The Docker socket is root-equivalent, and a read-only mount does not make the API read-only | A container holding the socket controls the Docker VM | Traefik uses the file provider; log discovery goes through a socket proxy that allows only read calls; the deploy agent is the only writer | P1-07, P2-06 |
| Memory and disk on a shared workstation are limited | The stack does not fit, or Docker's disk image fills the system drive | Per-container limits, profiles to run subsets, the full-stack footprint measured, Docker's disk image on a data drive | P1-01, P1-16 |
| VPN clients can route the same private ranges Docker uses | Containers lose connectivity while a VPN is up | Compose networks use explicit subnets outside common VPN ranges | P1-02 |
| Git for Windows converts LF to CRLF on checkout by default | Shell scripts and configs mounted into Linux containers break | `.gitattributes` forces LF (CRLF only for PowerShell scripts) | P0-02 |
| Backups live on the same machine | Losing the machine loses the lab | Accepted for a lab; a restore drill proves the backups work | P1-13, P1-14 |
| Lab keys, state and backups sit on a daily-use workstation | A stolen disk exposes them | BitLocker on the data drive; keys encrypted at rest | P1-01 |
| Measurements on a shared workstation are noisy | Numbers vary from run to run | Every result records hardware, versions, run count and the median | all |

## TLS and names

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| The lab root CA is trusted by the workstation's browser | A leaked CA key could impersonate any site | X.509 name constraints on the root and the intermediate limit them to the lab domain; the root key is stored encrypted outside the running stack | P1-04 |
| Java, Python, Go and Node each keep their own trust store | Clients fail, or someone disables verification to make them work | Trust injected per runtime; a CI check fails on disabled TLS verification | P1-05 |
| CI cannot use the host's CA | Tests differ from the host | CI creates a throwaway CA per run with the same layout | P1-03 |

## CI and supply chain

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| Rebase merges create commits without signature verification | Required signed commits and per-concern commits conflict | Merge commits: branch commits are signed by the author, merge commits by GitHub | P0-05, P0-06 |
| Public repository: untrusted pull requests | Secrets or runners exposed | GitHub-hosted runners only; no `pull_request_target`; actionlint and zizmor; secrets only in jobs on `main` | P0-16 |
| GitHub OIDC subject format: jobs with an environment use the environment form, and repositories created after 2026-07-15 carry immutable owner and repository IDs | AWS trust policies fail or are too broad | Trust policies written from an observed `sub` value | M10-07 |
| Signer identity tied to the repository name | A rename breaks verification; a re-created name could pass a name-only check | Verification also checks the repository ID in the signing certificate; the name is settled before the first signed build | P2-03 |
| Registry names must be lowercase | Pushes fail for a mixed-case owner | Workflows lowercase the owner | P2-01 |
| Docker Hub limits anonymous pulls | CI fails at random | Base images from registries without that limit, or authenticated pulls | P2-05 |
| Keycloak and other services migrate their schema on start and cannot downgrade | An image rollback fails or corrupts data | A backup before every migrating release; rollback across a migration means restore; our own services use expand-and-contract migrations | P2-08 |
| Third-party apps with write access to the repository | Supply-chain risk | Dependabot (first-party) instead of a third-party update app | P0-19 |
| Creating about 200 issues at once trips GitHub's secondary rate limits | The sync script stops halfway | Paced, idempotent, resumable sync keyed by plan ID | P0-08 |
| Scorecard checks a new, single-maintainer repository cannot pass yet (review by others, 90 days of activity) | The score is capped | The README reports the score with these known caps | P0-18 |
| Antivirus may flag unsigned Go binaries; Authenticode signing costs money | Install warnings on Windows | cosign signatures and checksums, a Scoop manifest, the Authenticode decision documented | M01-19 |
| Terraform is under the BSL 1.1 license | None for this use | Accepted; OpenTofu stays a drop-in fallback | M04-02 |

## AWS sandbox

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| With a workstation IdP as the identity source, nobody can sign in to Identity Center while the host is off | Lockout | A dedicated organization used only for the lab; tested break-glass access | P0-23, M12-06 |
| AWS might reject SAML metadata or URLs of a workstation-only IdP | Federation design fails late | Spike first, before the rest of M12 | M12-01 |
| Removing an Identity Center assignment does not end role sessions already issued | Access continues after revocation | Per-grant permission sets with an explicit deny after expiry and a one-hour session duration | M08-06 |
| Identity Center's SCIM support is partial (no nested groups, limited filters) | Connector assumptions break | Connector capability flags; Keycloak subgroups flattened; contract tests per capability | M07-04, M07-09 |
| Unexpected charges | Cost | Free services only; an SCP that allows only the services the lab uses; budget alarm | P0-23 |
| CloudTrail event history is per region and rate-limited | Missed events | Poll every region in use, within the limits | M13-03 |

## GitHub test organization

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| Synthetic users cannot have GitHub accounts (one free account per person) | Provisioning to GitHub cannot be tested at scale | Mocked GitHub API in CI; a nightly real test with the owner's account and one machine account | M07-08 |
| The Free plan lacks some features (for example branch protection on private repositories) | Rules fail on a free organization | Such rules report "not applicable" | M10-06 |
| A provisioning bug could remove the organization owner | Lockout | Safety rails: never remove owners or the last admin | M07-05 |

## Identity provider and Terraform

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| Terraform state can hold secrets in plain text | Leak through backups or git | Ephemeral resources and write-only arguments where the providers support them; disk encryption and encrypted backups for the rest; never in git | M04-02 |
| The Keycloak provider lags new Keycloak features | Some settings cannot be declared | Scripted, idempotent steps, each gap tracked as an issue | M04-09 |
| Changes made in an admin console drift from code | Reviews miss changes | Nightly plan must be empty; drift alert | M04-08 |
| Two writers for the same group memberships | Changes revert each other | One writer per object (see the Ownership table in PLAN.md) | M04-05, M07-05 |
| Removed roles stay in access tokens until they expire | Revocation lags in the apps | Short access tokens and back-channel logout; the lag is measured | M05-04, M08-13 |
| OpenSAML is not on Maven Central | The SAML app's build depends on another repository | Shibboleth repository pinned, with Gradle dependency verification | M05-02 |

## Workstation CLI

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| The OS keystore protects against other users and disk theft, not against malware running as the same user | Stored keys can be read by such malware | Documented in the threat model; short-lived credentials preferred over stored keys | M01-03 |
| credential_process runs on every AWS call | A slow or noisy provider breaks every tool | Answers from cache within a latency target; prints nothing but the credentials | M01-10 |
| SSO OIDC client registrations expire | Refresh stops working | Automatic re-registration | M01-08 |
| Windows shows notifications only for registered apps | Expiry warnings never appear | Spike early | M01-04 |
| Real account data in screenshots or recordings | Data leak | Demos use a demo configuration only | M01-20, R1-07 |

## Lab safety

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| The attack simulator (password spraying) could be pointed at a real system | Misuse | It refuses targets outside the lab domain and is rate-limited | M13-09 |
| Interactive chat approvals usually need a public callback URL | Breaks the this-machine-only rule | Slack Socket Mode or a Discord gateway bot (outbound connections only) | M08-11 |
| Credentials and certificates expire unnoticed (SCIM token, app keys, CA certificates) | Outages | Expiry inventory with alerts | M13-08 |
| A credential issued under a grant outlives it (SSH certificate, OpenBao token, AWS session) | Access after expiry | Lifetimes capped by the grant; tested per target | M08-07, M02-07, M03-03 |

## Parallel work

| Risk | Effect | Mitigation | Where |
|---|---|---|---|
| Shared files (`go.mod`, the base compose file, workflows, the plan) | Merge conflicts across tracks | Module-owned directories and compose files, path-filtered workflows; shared files change in their own pull requests | all |
| Only one stack can hold port 443 | Parallel sessions collide | One running stack per host; one worktree per track for code | all |
