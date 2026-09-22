# Security policy

## Reporting a vulnerability

Please do not open a public issue for a security problem.

Use GitHub's private vulnerability reporting on this repository
(**Security › Report a vulnerability**). This is a one-person hobby project:
reports are read and taken seriously, fixes are best effort, and there is no
guaranteed response time. You will be credited in the fix if you wish.

## What helps

Not required, but it speeds things up:

- the commit or release you tested;
- steps to reproduce, or a proof of concept;
- what an attacker gains (which data, which organisation boundary, which
  user), so the fix can be prioritised against the
  [invariants](docs/architecture/invariants.md).

## Scope

Anything in this repository: the server, the front end, the database
migrations, the container images, the `compose.yaml` used for self-hosting,
and the CI configuration.

## What is *not* a vulnerability

- Findings that require an already-compromised host or database.
- Running with a setting the documentation tells you to change before
  exposing the server (for example, a default password or a plain-HTTP
  listener without a TLS-terminating proxy in front).
