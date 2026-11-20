# P0 - Foundation

Track: all. Size: M. Depends on: nothing. Unlocks: everything.

## Goal

A repository that is safe to build in public from day one: rules, templates, decisions,
a threat model and CI security gates in place before the first line of product code,
plus the sandbox targets the early modules need.

## Scope

- Repository settings, governance files and rulesets.
- Project management: labels, milestones, a public project board, the plan-to-issues
  sync.
- The ADR process and the first ADRs.
- Threat model v1.
- CI skeleton with security gates.
- Developer tooling that works on Windows and Linux.
- The AWS sandbox organization and the GitHub test organization.

## Design notes

- Merge commits, not rebase merges: GitHub rewrites rebased commits without signature
  verification, which conflicts with required signed commits; squash merges would lose
  the per-concern commits.
- Git for Windows converts LF to CRLF on checkout by default; `.gitattributes` keeps
  every text file LF except PowerShell scripts.
- Commit signing and the SSH certificates of M02 both use the Windows OpenSSH agent;
  Git for Windows ships its own OpenSSH, so git is pointed at the Windows one.

## Issues

- P0-01 [chore] Repository files: LICENSE, SECURITY.md, CONTRIBUTING.md, CODE_OF_CONDUCT.md, CODEOWNERS -- Done when: merged; the license is chosen before the first code; SECURITY.md points to GitHub private vulnerability reporting.
- P0-02 [chore] .gitattributes and .gitignore -- Done when: LF is enforced for every text file except PowerShell scripts, binaries are marked, state files, keys, .env files and build output are ignored, and a fresh checkout on Windows with autocrlf=true keeps LF.
- P0-03 [security] Repository security settings -- Done when: private vulnerability reporting, secret scanning with push protection and Dependabot alerts are on; the default Actions token is read-only; workflows from fork pull requests need approval.
- P0-04 [chore] Commit signing -- Done when: the Windows OpenSSH agent service is enabled, git uses Windows OpenSSH for signing, an SSH signing key is registered on GitHub, and commits show as Verified.
- P0-05 [security] Rulesets for main and release tags -- Done when: main requires pull requests, required status checks and signed commits, allows merge commits only, and blocks force pushes and deletion; only the owner can create or delete v* tags; enabled after P0-04.
- P0-06 [adr] Merge strategy and signed commits -- Done when: accepted, explaining why rebase and squash merges are not used.
- P0-07 [chore] Labels, milestones and public project board -- Done when: type and area labels exist, one milestone per plan file, and a public project lists every issue, with saved views by milestone, track and phase.
- P0-08 [chore] Plan-to-issues sync script -- Done when: it reads docs/plan/*.md, creates or updates issues by plan ID with labels and milestone, adds them to the board, paces requests under GitHub's secondary rate limits, and resumes after a failure.
- P0-09 [chore] Issue and pull request templates -- Done when: feature, bug, spike and ADR issue templates; a pull request template with the definition-of-done checklist.
- P0-10 [adr] Record architecture decisions -- Done when: template in `docs/adr/` and an index that CI keeps current.
- P0-11 [adr] Host on Docker Desktop, this machine only -- Done when: accepted, with the alternatives considered.
- P0-12 [adr] Pull-based signed delivery -- Done when: accepted, with the threat it removes (no runner on the host for a public repository).
- P0-13 [adr] Languages per component -- Done when: accepted: Go for services and tools, Spring Boot and FastAPI demo apps, React portal.
- P0-14 [adr] Monorepo, one Go module, one version line -- Done when: accepted, with the release consequences for the CLI.
- P0-15 [security] Threat model v1 -- Done when: `docs/threat-model/` has the system context, trust boundaries, data flows and a STRIDE table for the platform and the CI/CD path, linked to RISKS.md, with an issue for every open risk.
- P0-16 [ci] CI skeleton -- Done when: every pull request runs actionlint, zizmor, gitleaks, an ASCII guard, a Conventional Commits check and dependency review; every action is pinned by commit SHA; workflow permissions default to read-only; no workflow uses pull_request_target, and untrusted input never reaches a run step directly.
- P0-17 [ci] Code scanning -- Done when: CodeQL runs on pull requests and weekly, with each language (Go, Java, Python, TypeScript) enabled when its first code lands.
- P0-18 [ci] OpenSSF Scorecard -- Done when: a weekly run publishes results to code scanning and a README badge, and the README notes the checks a new single-maintainer repository cannot pass yet.
- P0-19 [ci] Dependabot -- Done when: action SHAs, Dockerfile and Compose image digests, Terraform providers, and Go, Java, Python and npm dependencies get grouped weekly pull requests.
- P0-20 [ci] License check -- Done when: dependency licenses are checked against an allow list in CI and third-party notices are generated.
- P0-21 [ci] Runner hardening -- Done when: StepSecurity harden-runner runs on every job, first in audit mode, then blocking with an egress allow list.
- P0-22 [chore] Developer tooling -- Done when: a Taskfile with lint, test and up tasks and pre-commit hooks for formatting and gitleaks work on Windows and Linux, with git long paths enabled on Windows.
- P0-23 [chore] Sandbox AWS organization -- Done when: a new AWS Organization used only for the lab has IAM Identity Center, a budget alarm at USD 1, centralized root access management for member accounts, an SCP allowing only the services the lab uses, and a management account root user with MFA and no access keys.
- P0-24 [chore] GitHub test organization -- Done when: a free organization with public test repositories exists, plus one machine account for tests and least-privilege credentials for idaudit and provisiond.

## Measures

- OpenSSF Scorecard score at the end of P0 (baseline, with the known caps noted).
