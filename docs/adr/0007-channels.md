# 0007. Channels: public or private, explicit members, one organisation

Date: 2026-09-22, revised 2026-09-23 for the flat model (ADR 0004) and
simplified after review the same day. Status: accepted.

## Context

In the flat model a channel is the only grouping of people, so isolation
between stores is a matter of which channels exist and who is in them.
Channel membership must therefore be explicit and easy to reason about.

The previous version designed for sharing a channel with another
organisation (Slack Connect) from the start, which meant authorising the
whole channel subtree through membership *instead of* the tenant predicate,
plus an `owner_organization_id` column with special semantics. That was the
single largest source of remaining complexity, for a feature the README
itself lists as "later".

## Decision

- Every channel belongs to exactly one organisation, and **the organisation
  is the absolute boundary**: channel rows and everything under them
  (members, topics, messages, attachments, reactions, read state) carry
  `organization_id` and are scoped by it like any other tenant-owned data
  (invariant I-1, no exception). Access to a channel's content additionally
  requires **membership** of that channel.
- A channel is **public** — any user of the organisation can discover it
  and create their own membership (self-join) — or **private** — visible
  only to its members; membership is obtained by invitation or as an
  initial membership during user onboarding (ADR 0005). Membership rows
  follow these policies; channel *content* is for members only (invariant
  I-2). Discovering public channels does not depend on `view_directory`;
  that capability is only about seeing people. A consequence for
  operators: a store is a *private* channel (a public one would make its
  members visible to everyone who joins), while organisation-wide channels
  such as announcements are public.
- Membership is a row per actor, with one flag: `is_admin`. An admin of a
  channel can rename it, invite and remove members and install bots there.
  There is no per-channel capability system beyond this flag.
- People with the organisation-wide capability create channels.
- A channel spanning two stores is simply a channel with members from both.
  Membership of any channel — including an organisation-wide public one —
  intentionally creates reachability between its user members (ADR 0005).
- A topic is never moved to a channel of a different organisation — which
  under the absolute boundary is simply I-1 again, kept as I-10 for the
  move operation's own test.
- Reachability's shared-channel clause reads channel membership (ADR 0005).

## Sharing with another organisation, later

Cross-organisation sharing is deliberately not designed here. Membership is
expected to remain the useful authorisation seam, but a future ADR must
define the tenant boundary, schema, read and write paths, attachments,
reactions, read state, search, notifications, events, audit and export
rules before any implementation.

## Consequences

- Invariants I-1 applies uniformly; I-2 now states the membership
  requirement; I-7 and I-10 as above.
- No `owner_organization_id` column. No channel scopes.
