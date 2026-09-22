# 0008. Username and password authentication, credentials separated from users

Date: 2026-09-22. Status: accepted.

## Context

Target users include part-time staff who may have no work email address.
Business deployments will eventually want OIDC (Google Workspace, Microsoft
Entra), but adding it in v1 multiplies the account-linking and invitation
issues before the core exists.

## Decision

- v1 signs in with **username and password**. The username is unique **within
  an organisation** and **may not contain `@`**, so an email address can never
  be used as a login id. Email is optional and exists for recovery.
- Every actor has an **internal id** (`actor_id`, opaque, e.g. UUID) that is
  the only identifier for a person or bot allowed in logs; usernames and
  emails are not logged.
- Recovery:
  - forgot username, email on file → the username is sent to that email;
  - forgot password, email on file → self-service reset by email;
  - no email on file → an administrator can look up the username, issue a
    login aid such as a QR code, and reset the password. **Every
    administrative reset writes an audit record and notifies the user and
    the organisation's administrators.** *Who may reset whose credential is
    not yet decided* and must be before the authentication issue starts:
    capabilities are organisation-wide (ADR 0004), so "holds the reset
    capability" alone would let a Shinjuku manager reset an Osaka
    part-timer. Candidates: an action policy
    `canAdminResetCredential(actor, target)` requiring the capability *and*
    `canReach(actor, target)`; or granting the capability only to
    organisation-wide administrators. Who is notified is defined the same
    way — by capability, for example every user holding the
    administrative-reset capability — because there is no fixed
    "administrator" role name to point at. A login aid that carries a
    credential (rather than just the username) is treated like an invite
    link: short-lived, single-use, never logged.
- Later: OIDC providers including Google and LINE (LINE Login is
  OIDC-compatible).
- Passwords are hashed with Argon2id. Sign-in is rate limited. Sessions are
  server-side and referenced by an HttpOnly, SameSite cookie; they can be
  revoked.
- Credentials live in a `credential` table (kind + secret), not on the
  actor, so OIDC and API tokens (ADR 0012) are additional credential kinds
  rather than schema changes.
- End-to-end encryption is out of scope: it conflicts with the administrative
  oversight the product exists to provide.

## Consequences

- Because usernames are per-organisation, sign-in must identify the
  organisation. Sub-domains are a poor fit for people running the container
  at home, so the candidates are a path prefix (`/o/{slug}/...`) or an
  organisation field on the sign-in form. **Decided in the authentication
  issue** (Definition of Ready).
- The scope of administrative credential reset (above) is part of the
  authentication issue's Definition of Ready.
- The authentication issue is the first `risk:high` issue and the point at
  which Jules is configured (ADR 0009).
