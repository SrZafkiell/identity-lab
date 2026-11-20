# M02 - SSH certificate authority

Track: A. Size: M. Depends on: M04. Unlocks: SSH principals granted through JIT (M08).

## Goal

No long-lived SSH keys on servers: people log in with SSO and get an SSH certificate that
expires in hours, and hosts get certificates too, so there is no trust-on-first-use
prompt.

## Design notes

- ADR: step-ca (OIDC provisioner, SSH certificate templates) vs an own signer built on
  `golang.org/x/crypto/ssh`. Working choice: step-ca, because CA code is
  security-critical; the value here is the integration and the policy, not a new CA.
- Principals come from Keycloak groups through the certificate template. Privileged
  principals come only through JIT (M08), and their certificates never outlive the
  grant; serials of expired grants go to the revocation list.
- Lab targets: a bastion and two target containers whose sshd trusts the user CA and
  presents host certificates; optionally the owner's own Linux VMs.
- The CLI wraps the login and loads the certificate into the SSH agent. On Windows that
  is the Windows OpenSSH agent; Git for Windows ships a separate OpenSSH whose agent is
  not used.

## Issues

- M02-01 [adr] SSH CA choice -- Done when: accepted, with key storage and rotation covered.
- M02-02 [feature] step-ca service in the `ssh` profile -- Done when: CA keys are stored encrypted with the password from SOPS, a key backup procedure exists, and the service has a health check.
- M02-03 [feature] OIDC provisioner with Keycloak -- Done when: an SSO login issues a user certificate valid for 8 hours with principals from groups.
- M02-04 [feature] Host certificates -- Done when: lab hosts get host certificates at start-up and renew them, and clients trust the host CA with no known_hosts prompts.
- M02-05 [feature] Lab SSH targets -- Done when: the bastion and targets accept only CA-signed certificates, password and plain key auth are off, and ProxyJump through the bastion works.
- M02-06 [feature] CLI: `idlab ssh login` and `idlab ssh config` -- Done when: login gets the certificate, loads it into the Windows OpenSSH agent (or ssh-agent elsewhere) with a matching lifetime, and writes an ssh_config include for the lab hosts.
- M02-07 [security] Privileged certificates follow JIT -- Done when: certificates with privileged principals never outlive the JIT grant, and serials of expired grants are on the revocation list (tested).
- M02-08 [security] Revocation -- Done when: a revoked certificate is refused, with the method (KRL or passive revocation) documented and tested.
- M02-09 [feature] Issuance audit -- Done when: every issued certificate (user, principals, serial, validity) is logged and shipped to Loki.
- M02-10 [test] Tests -- Done when: an integration test logs in, connects through the bastion, and fails after expiry and after revocation.
- M02-11 [docs] Runbook -- Done when: CA key rotation and host onboarding are documented.

## Measures

- Certificate lifetime and time from login to the first SSH connection.
- Static `authorized_keys` entries on lab hosts (target 0, checked by a test).
