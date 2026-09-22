# 0012. Public API and integrations from the start

Date: 2026-09-22. Status: accepted.

## Context

The maintainer intends to offer a REST API to third parties and to accept
data from outside through webhooks. Bolting these on later is expensive in
exactly the places that are hard to change: who authors a message, how a
caller authenticates, and whether the front end depends on endpoints the
public API does not have.

## Decision

- **One API.** Everything is served under a versioned prefix (`/api/v1`),
  described by the OpenAPI document, and the single-page app is an ordinary
  client of it. There are no front-end-only endpoints. If the SPA needs
  something, the public API gets it.
- **Actors.** The thing that posts a message, is a member of a channel,
  and appears in an audit record is an `actor`. A `user` is a human
  actor with sign-in credentials; a `bot` is an actor owned by an
  organisation for integrations. Authorship, channel membership and
  audit reference actors, never users directly; roles and reachability
  apply to users only. **Every actor belongs to exactly one organisation,
  fixed at creation**; a person active in two organisations has two
  actors. `token → actor → organization` always resolves to one
  organisation.
- **Bots are channel members only, with fixed permissions.** A bot is
  installed into a channel by `canInstallBot(actor, bot, channel)`: the
  actor is an admin of that channel or holds the organisation-wide
  capability, and the bot belongs to the same organisation. For v1 a bot:
  - holds no roles and no organisation-wide capabilities;
  - may be installed into a channel;
  - may post only in channels it is installed in;
  - may not DM, create channels, invite users, move messages, or perform
    any other organisation-wide or administrative action.

  A bot has no directory visibility and is outside reachability.
  Installation is its own policy, not an invitation. An API token scope may
  narrow these permissions further but never extends them. If a bot ever
  needs more, that is a new requirement to design, not a role to grant.
- **API tokens** are a credential kind (ADR 0008). A token belongs to one
  actor, so to one organisation; its scope can only narrow what its actor
  is otherwise allowed to do (a user's capabilities, a bot's fixed
  permissions) and never grants more; it is stored hashed and never
  logged. Bots authenticate to the general API only with tokens.
- **Every route declares its allowed authentication modes** in the route
  table: one or more of `session` (cookie, Origin-checked, CSRF-protected),
  `token` (API token) and `webhook` (per-integration secret), or `public`
  on its own — `public` is never combined with another mode. The
  route-table sweep generates the wrong-mode cases for each.
- **Incoming webhooks** are a per-webhook secret bound to a bot and a
  channel (and optionally a topic). The external system presents the
  credential **in a request header** — never in the URL, where it would
  land in proxy and access logs, traces and copied links. Whether that
  header carries the secret itself or an HMAC signature is decided in the
  integrations issue. The server authenticates the *webhook* and then acts
  as its bound bot. Webhook secrets are never logged. The bot's own API
  tokens are a separate credential for the general API. Posts are subject
  to the same membership rules as any actor, with payload size limits and
  rate limits.
- **Outgoing webhooks and other event consumers** subscribe to the domain
  events of ADR 0006. Events therefore have stable names and payloads from
  the first one.
- The WebAssembly plugin sandbox of ADR 0010 is a third integration surface
  and shares actors, tokens and events with the two above.

## Consequences

- The first schema has `actor`, `user` and `bot` rather than `user` alone.
- The API issue's Definition of Ready includes: error format (RFC 9457
  `application/problem+json`), cursor pagination, idempotency keys on
  non-idempotent writes, rate limiting, a versioning and deprecation
  policy.
- Invariants I-20 to I-23.

## Open questions

- Whether the webhook header carries a shared secret or an HMAC signature.
  If HMAC: the signature must cover a timestamp and the body, and the issue
  must define the timestamp tolerance, replay detection and the handling
  of duplicate requests. Integrations issue.
