# R1 - v1.0.0 release

Track: all. Size: M. Depends on: every module.

## Goal

A release someone can clone, run and understand in one sitting, with evidence for every
claim.

## Issues

- R1-01 [security] Threat model and risk review -- Done when: the threat model and RISKS.md reflect what was built, and every open risk is fixed or accepted with a reason.
- R1-02 [security] Final scan pass -- Done when: CodeQL, Trivy, ZAP and idaudit on the lab configuration show no open high findings.
- R1-03 [docs] Quick start -- Done when: a clean machine goes from clone to a working login by following the documented steps, and the time is recorded.
- R1-04 [docs] Architecture documentation -- Done when: diagrams, module guides, the ADR index and the runbooks are complete.
- R1-05 [docs] Results page -- Done when: every number in `docs/results/` comes from a script or test run, with its date, command, hardware and versions.
- R1-06 [docs] Documentation site (optional) -- Done when: static documentation is published on GitHub Pages.
- R1-07 [chore] Walkthrough video -- Done when: a short video recorded on demo data only shows login, MFA, JIT, a review, an audit, an alert and a deploy.
- R1-08 [chore] OpenSSF Best Practices badge -- Done when: the passing level is reached and linked from the README.
- R1-09 [chore] Tag v1.0.0 -- Done when: a signed release with images, binaries, SBOMs and release notes is published.

## Measures

- Time from clone to first login on a clean machine.
