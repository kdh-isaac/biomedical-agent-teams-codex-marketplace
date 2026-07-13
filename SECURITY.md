# Security Policy

This repository packages the `biomedical-agent-teams` (BMAT) Codex plugin: a
prompt-and-script bundle (agent prompts, command recipes, contract schemas,
and deterministic Python validators). It does not run a hosted service and
does not process third-party user data.

## Scope

Security-relevant issues in this repository generally fall into one of these
categories:

- A `scripts/bmat_*.py` validator that fails to fail-closed (accepts an
  artifact bundle it should reject, e.g. bypassing the self-attestation,
  empty-evidence, or fabricated-identifier checks described in
  `references/biomedical-failure-modes.md`).
- A prompt or template that could cause an agent to exfiltrate private or
  PHI-adjacent context to a public-only connector or loop
  (`references/connector-binding-matrix.md`, `references/data-safety-floor.md`).
- Supply-chain issues in the repository itself (malicious or unpinned
  dependency additions, tampered release artifacts).

This project intentionally has no runtime network service, no stored
credentials, and no user database, so classic web/application vulnerability
classes (XSS, SQLi, auth bypass, etc.) do not apply.

## Reporting a Vulnerability

Please open a private security advisory on the GitHub repository
(`kdh-isaac/biomedical-agent-teams-codex-marketplace` -> Security -> Report a
vulnerability) rather than a public issue. Include:

- The affected file(s) (validator script, contract schema, or prompt file).
- A minimal artifact bundle or input that reproduces the problem.
- What the validator currently does versus what it should do.

There is no fixed SLA; this is a single-maintainer research-workflow project.
Reports will be acknowledged and triaged as soon as practical.

## Supported Versions

Only the latest tagged release on the `main` branch is supported. Older
versions do not receive backported fixes.
