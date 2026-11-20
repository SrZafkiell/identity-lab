# P1 - Platform runtime

Track: B. Size: L. Depends on: P0. Unlocks: M04, P2, M13.

## Goal

The base every module runs on: a modular Docker Compose stack with a reverse proxy,
trusted TLS from a lab CA, one PostgreSQL, secrets that never sit in plain text, backups,
logs and metrics.

## Design notes

- One compose file per module, joined with `include` and started by profile (`core`,
  `apps`, `ssh`, `secrets`, `governance`, `monitoring`, `portal`), so any module can run
  alone on top of `core`. Networks use explicit subnets that avoid common VPN ranges.
- This machine only: only Traefik publishes ports (80 and 443), bound to 127.0.0.1.
- TLS: a lab root CA and an intermediate, both with X.509 name constraints limited to
  the lab's `.internal` domain, so a leaked key cannot impersonate other sites. The root
  key is stored encrypted outside the running stack. Hosts-file entries point the lab
  names at 127.0.0.1, and Traefik carries the same names as network aliases inside
  Docker, so browsers and containers use the same URLs. CI builds a throwaway CA per
  run with the same layout. step-ca (M02) can later take over issuance through ACME.
- Every runtime gets the lab CA in its own trust store (Java, Python, Go, Node); nothing
  disables TLS verification.
- Docker socket: Traefik uses its file provider, so it needs no socket; log discovery
  goes through a socket proxy that allows only read calls; only the deploy agent (P2)
  has write access. A read-only mount of the socket does not make the API read-only.
- Hardening baseline for every container: pinned digest, non-root user, read-only root
  file system where possible, no new privileges, dropped capabilities, CPU and memory
  limits, health check.
- The WSL2 clock can drift after the host sleeps, which breaks TOTP, token and assertion
  validity, AWS request signing and signature checks; a health check watches the skew.
- Bootstrap secrets live in SOPS-encrypted files; the age key exists only on the host.

## Issues

- P1-01 [chore] Host preparation -- Done when: the WSL2 memory limit is set, Docker Desktop's disk image is on a drive with room, the drive holding lab data uses BitLocker, and the start-at-sign-in choice is documented.
- P1-02 [feature] Compose skeleton with module profiles -- Done when: `task up PROFILE=core` starts the proxy and PostgreSQL, each module has its own compose file joined with include, networks use explicit subnets, and every profile is documented.
- P1-03 [adr] Lab TLS: name-constrained internal CA -- Done when: accepted, covering the `.internal` lab domain, hosts-file entries, network aliases inside Docker, CA trust per runtime and the throwaway CA in CI.
- P1-04 [feature] Lab CA -- Done when: a root and an intermediate with name constraints limited to the lab domain exist, the root key is stored encrypted outside the stack, the root is installed in the Windows trust store, and a task renews the wildcard certificate.
- P1-05 [security] CA trust in every runtime -- Done when: Java, Python, Go and Node clients in the lab trust the lab CA, and a CI check fails on disabled TLS verification (InsecureSkipVerify, verify=False, NODE_TLS_REJECT_UNAUTHORIZED=0 and similar).
- P1-06 [feature] Reverse proxy -- Done when: Traefik uses the file provider, publishes only ports 80 and 443 on 127.0.0.1, terminates TLS with the lab certificate, redirects HTTP to HTTPS, sets security headers (HSTS, CSP default, frame options) and keeps its dashboard admin-only.
- P1-07 [security] Docker socket access -- Done when: no container mounts the Docker socket except the deploy agent, log discovery uses a socket proxy limited to read calls, and a conftest rule enforces both.
- P1-08 [feature] PostgreSQL -- Done when: one database and one role per service, created by an init script; no service uses the superuser; data on a named volume.
- P1-09 [security] Container hardening baseline -- Done when: a conftest policy in CI fails on a missing user, missing limits, privileged mode, an image without a digest, or a port published on anything but 127.0.0.1.
- P1-10 [adr] Bootstrap secrets with SOPS and age -- Done when: accepted, including how secrets move to OpenBao once M03 lands.
- P1-11 [feature] Secret handling -- Done when: secrets are decrypted only at deploy time into tmpfs or Docker secrets, gitleaks finds nothing in history, and rotation is documented.
- P1-12 [feature] Clock skew check -- Done when: a health check compares container time with the host, alerts above 2 seconds, and the fix after sleep is documented.
- P1-13 [feature] Backups -- Done when: nightly encrypted restic backups of PostgreSQL and volumes go to a second drive with a retention policy.
- P1-14 [test] Restore drill -- Done when: a script restores into a clean stack and verifies the data; the restore time is recorded in `docs/results/`.
- P1-15 [feature] Observability base -- Done when: Alloy ships container logs to Loki, Prometheus scrapes every service, and Grafana is provisioned from code with a platform dashboard.
- P1-16 [test] Resource budget -- Done when: memory and CPU of the full stack are measured and stay within the WSL2 limit with headroom.
- P1-17 [docs] Runtime runbook -- Done when: start, stop, upgrade, reset and troubleshooting are documented.

## Measures

- Restore time (RTO) and backup interval (RPO).
- Cold start time of the `core` profile; memory of the full stack.
