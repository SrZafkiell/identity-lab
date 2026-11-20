# M01 - AWS access manager

Track: A. Size: L. Depends on: P0. Unlocks: JIT requests from the CLI (M08), sandbox
login through Keycloak (M12).

## Goal

One CLI with a TUI to manage AWS CLI profiles and SSO sessions across several
organizations: add and remove accounts, save SSO start URLs, log in from the tool, keep
sessions fresh in the background and open the console for any account. It works with
any AWS SSO setup, not only this platform, so it is useful from its first release.

## Features

Profiles and accounts
- Add, edit, rename, remove and list profiles: SSO, role chains and static keys.
- Writes `~/.aws/config` safely: comments and order kept, a backup before every change,
  atomic writes, a file lock, and `--dry-run` that shows the diff.
- Imports existing profiles; finds duplicates and broken `source_profile` chains.
- Naming templates for generated profiles, for example `{org}-{account}-{role}`.

SSO sessions
- Saves SSO start URLs and regions as named sessions (`sso-session` blocks).
- `idlab aws login <session>` starts the browser login (authorization code with PKCE,
  device code as fallback).
- Discovers every account and role a session can reach and creates the profiles;
  accounts that disappeared are flagged, never deleted silently.
- Several sessions at once, one per organization.

Background refresh
- Agent mode starts at logon (Task Scheduler on Windows, launchd on macOS, a systemd
  user unit on Linux).
- Refreshes the SSO access token with its refresh token before it expires, and keeps
  role credentials warm for profiles marked for it.
- When the Identity Center session itself ends (it cannot be extended past the
  organization's session limit), sends a desktop notification with a one-click
  re-login.

Using credentials
- `credential_process` integration, so every AWS SDK and tool gets credentials through
  idlab without plain-text credential files.
- `idlab aws exec <profile> -- <command>` and `idlab aws env <profile>` for PowerShell,
  bash, zsh and fish.
- Static keys, when unavoidable, live in the OS keystore (Windows Credential Manager,
  macOS Keychain, Secret Service), with MFA sessions through `GetSessionToken`.
- A shell prompt segment shows the active account and the time left.

Console
- `idlab aws console <profile>` opens the AWS console for that account and role through
  the federation sign-in endpoint, optionally in a browser profile per organization.

TUI
- Sessions and profiles with status and expiry; login, refresh, console and exec from
  the keyboard.

Stretch
- `idlab aws ecr-login` and kubeconfig entries per profile for EKS.
- Export and import of the session list, without secrets, to move between machines.

## Design notes

- `idlab` is a working name; the final CLI name is chosen before the first release
  (IDLab is also the name of a well-known research group).
- Prior art: aws-vault, Granted, aws-sso-util. What is different here: Windows first;
  one tool for sessions, keystore, refresh and console; and JIT requests (M08) from the
  same CLI.
- The SSO token cache stays compatible with the AWS CLI, so `aws` and idlab share
  sessions. The AWS CLI does not lock that cache, so every write is atomic (write, then
  rename).
- SSO OIDC client registrations expire (about every 90 days for the AWS CLI's own);
  idlab re-registers before that happens.
- `credential_process` runs on every AWS call, so it must answer from cache, fast, and
  print nothing but the credentials JSON.
- The OS keystore has size limits (Windows Credential Manager holds at most 2,560 bytes
  per secret), and it protects against other users and disk theft, not against malware
  running as the same user. Short-lived credentials are preferred over stored keys.
- Windows shows notifications only for apps registered with an AppUserModelID; an early
  spike proves this works for an unpackaged binary.
- Windows antivirus may flag unsigned Go binaries, and Authenticode signing costs
  money; releases carry cosign signatures and checksums, and the Authenticode choice is
  documented.
- Demos and recordings use a demo configuration only, never real accounts.
- Bubble Tea for the TUI; AWS SDK for Go v2 for SSO, SSO OIDC and STS.

## Issues

- M01-01 [chore] Choose the CLI name -- Done when: a name is picked, checked for clashes with existing tools, and used everywhere before the first release.
- M01-02 [adr] CLI architecture: commands, agent mode, local IPC -- Done when: accepted; the CLI and the agent talk over a named pipe on Windows or a Unix socket elsewhere, owner-only.
- M01-03 [adr] Secret storage on the workstation -- Done when: accepted, with the keystore size limits, the fallback (DPAPI or an age-encrypted file) and what the keystore does and does not protect against.
- M01-04 [spike] Desktop notifications on Windows -- Done when: a notification from the unpackaged binary shows reliably and clicking it runs a command.
- M01-05 [feature] Config store for ~/.aws/config -- Done when: comments and order survive round trips; atomic write, backup, lock and dry-run diff work; the parser has a fuzz test.
- M01-06 [feature] Profile commands: add, edit, rename, remove, list, import -- Done when: all profile types are supported and validation catches broken role chains.
- M01-07 [feature] SSO sessions: save, list, remove -- Done when: `sso-session` blocks are managed and the start URL and region are validated.
- M01-08 [feature] SSO login -- Done when: authorization code with PKCE works against a real Identity Center, device code works as fallback, expired client registrations are renewed automatically, and cache writes are atomic and compatible with the AWS CLI.
- M01-09 [feature] Account and role discovery -- Done when: profiles are created from naming templates and removed accounts are flagged.
- M01-10 [feature] credential_process provider -- Done when: `aws sts get-caller-identity --profile <name>` works through idlab for SSO, role-chain and keystore profiles; a cached answer takes under 100 ms and prints nothing but the credentials JSON.
- M01-11 [feature] exec and env commands -- Done when: output is correct for PowerShell, bash, zsh and fish, and credentials never appear in logs.
- M01-12 [feature] Static keys in the OS keystore with MFA sessions -- Done when: keys are stored only in the keystore and MFA sessions are cached until they expire.
- M01-13 [feature] Background agent with refresh -- Done when: it installs as a logon task, refreshes SSO tokens before expiry, keeps marked profiles warm and survives sleep and resume.
- M01-14 [feature] Expiry notifications with one-click re-login -- Done when: a desktop notification arrives before the session ends and clicking it starts the login.
- M01-15 [feature] Console sign-in -- Done when: the console opens for any SSO or role profile and the sign-in URL is never logged.
- M01-16 [feature] TUI -- Done when: list, status, expiry, login, refresh, console and exec are reachable from the keyboard.
- M01-17 [feature] Shell prompt segment -- Done when: snippets for PowerShell and bash or zsh show the profile and the time left.
- M01-18 [test] Test suite -- Done when: unit tests run against a fake AWS endpoint, config writes have golden files, and an integration test runs against the sandbox.
- M01-19 [ci] Release binaries -- Done when: Windows, macOS and Linux builds ship with checksums, cosign signatures and an SBOM; a Scoop manifest exists; the Authenticode decision and antivirus false-positive reporting are documented.
- M01-20 [docs] User guide -- Done when: install, first login, discovery, the refresh agent, the console and troubleshooting are documented, with a demo recording made from a demo configuration.

## Measures

- credential_process latency for cached answers (p50 and p95).
- Token refresh success rate over a week of daily use (from the agent log).
- Time from command to an open console session.

## Out of scope

- Storing or syncing anything in a cloud service.
