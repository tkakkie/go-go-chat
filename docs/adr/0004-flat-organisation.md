# 0004. A flat organisation: no hierarchy in authorisation

Date: 2026-09-23. Status: accepted. Replaces the group-tree design of the
same number written on 2026-09-22, before anything was implemented.

## Context

The target users are organisations with structure — head office, areas,
stores — where a part-timer in one store must not see or message the staff
of another, and managers must reach the people they manage. The first design
modelled that structure directly: a group tree per organisation, channels
bound to groups, capabilities inherited down the tree, reachability derived
from ancestry, channel eligibility through subtrees.

Reviewing it, the maintainer judged the authorisation model to have become
larger than the problem: a closure table, scoped and inherited capabilities,
directional reachability reasons, subtree eligibility and an intra-
organisation "shared channel" concept, all before a message could be sent.
None of that is the project's learning goal, and it is not what a portfolio
reader will admire.

Zulip, the reference product, has no hierarchy. It isolates people with four
simpler things: organisation-level roles; *guests* who can see only the users
they share a channel with; organisation settings for who may start direct
messages and who may create channels; and user groups — flat sets, possibly
nested — for bulk membership. Its isolation primitive is "we share a
channel", nothing more.

## Decision

- **There is no group tree.** An organisation contains users, channels and
  settings. Nothing in authorisation depends on where a user sits in a
  hierarchy, because there is none.
- **Isolation comes from channel membership and directory visibility.** A
  user who holds the `view_directory` capability sees every user in the
  organisation; a user without it (the ordinary part-timer) sees only users
  who share a channel with them. A store is therefore a channel, or a few;
  people in different stores never see each other unless a channel joins
  them.
- **Roles are organisation-wide.** A role is a named bundle of capabilities
  defined per organisation and granted to a user for the whole
  organisation. A capability has no scope other than the organisation.
  Channel-local authorisation is a single `is_admin` flag on the channel
  membership, which is not a capability. Effective capabilities are the
  union of all roles; there is no deny and no priority.
- **Restrictions are settings, not structure.** Who may start a direct
  message, with whom, who may create channels, and any per-user exceptions
  ("this person may message that person") are organisation settings and
  action policies (ADR 0005), not tree positions.
- **Bulk management comes later, as a convenience.** If administering a
  large chain by hand proves painful, a *user group* concept — a flat list
  of users, not nested — will be added for adding many people to a channel
  at once. Nesting is not planned; it would need cycle detection and
  recursive expansion for a use case nobody has yet. It will never be an
  authorisation primitive; it only expands to individual memberships.
  Store-per-channel templates are a similar convenience, added if wanted.

## Consequences

- Reachability (ADR 0005) collapses to one boolean: share a channel, or
  hold `view_directory`. No closure table, no inheritance, no subtree logic.
- Channels (ADR 0007) belong to the organisation, not to a group; a channel
  spanning two stores is just a channel. The organisation is the absolute
  boundary; cross-organisation sharing is not designed in.
- Invariants I-5 and I-11 of the tree design are withdrawn; I-7 and I-22 are
  restated.
- Administering a hundred stores means a hundred channels and their
  memberships, until user groups exist. Accepted.
