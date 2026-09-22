# 0005. Reachability as a primitive, authorisation as per-action policies

Date: 2026-09-22, revised 2026-09-23 for the flat model (ADR 0004) and
simplified after review the same day. Status: accepted.

## Context

"Who can see and message whom" is the product's core privacy requirement.
Part-timers in one store should not see staff of another; some people must
be able to reach everyone; an organisation may forbid direct messages between
ordinary members. The same question is asked by DMs, group DMs, channel
invitations, user search and mentions.

Earlier versions returned a *set of reasons* from reachability so that a
policy could treat "reachable through a shared channel" differently from
"reachable through the directory". Under the flat model no such policy
exists: every rule that was written down turned out to depend on the
capabilities of the two people and on organisation settings, never on *why*
they can see each other. The set was complexity without a customer.

## Decision

**Reachability** is one boolean function in one package, defined between two
**users** (bots are outside it):

```text
canReach(a, b) = a and b share at least one channel
              or a holds the view_directory capability
```

It is directional: a manager with `view_directory` reaches a part-timer; the
part-timer reaches the manager only if they share a channel. It is the only
place that computes visibility, and it is an input to policies, never a
permission by itself.

A consequence to state plainly: **membership of any channel creates
reachability between its user members**, including organisation-wide public
channels such as announcements. Staff from two stores who both join
`#announcements` can see each other. That is intended; if an organisation
ever needs a broadcast channel that does not connect its readers, that is a
new concept to design then, not now.

**Action policies** are separate per-action authorisation functions. Each
uses only the inputs relevant to that action — reachability,
organisation-wide capabilities, channel membership, channel admin status,
organisation settings — and no policy is obliged to consult reachability:

- `canViewUser(a, b)` — reachability.
- `canInviteToChannel(actor, target, channel)` — the actor can reach the
  target, and the actor either holds the organisation-wide invite
  capability or is an admin (`is_admin`) of that channel.
- `canInstallBot(actor, bot, channel)` — the actor either holds the
  organisation-wide capability or is an admin of that channel, and the bot
  belongs to the same organisation. No reachability involved.
- **Onboarding is not invitation.** A brand-new user shares no channel with
  anyone, so nobody without `view_directory` could reach them and the
  invitation policy could never fire. Creating a user is therefore its own
  operation, guarded by an organisation-wide capability (working name
  `manage_users`), and it may set the new user's **initial channel
  memberships** in the same step: `canOnboardUserToChannel(actor, invitee,
  channel)` holds when the actor may create users and either holds the
  organisation-wide invite capability or is an admin of that channel.
  Reachability is not consulted; the invitee is new. From then on the
  ordinary invitation policy applies. `canReach` gets no exception for
  this. **Creating the user and all requested initial memberships is one
  transaction**: if any membership is refused — including because the
  actor lacks the right for one of the channels — nothing is created
  (invariant I-24).
- `canStartDM` and the other direct-message policies are **not settled**;
  see the open questions.

**Roles** are named bundles of capabilities, defined per organisation and
granted to a user for the whole organisation. A user may hold several;
effective capabilities are their union; there is no deny and no priority.
Channel-level rights are a single flag on the membership (`is_admin`), not a
second capability system. Access decisions ask for a capability, never a
role name.

## Consequences

- Invariants I-3, I-4, I-6, I-7, I-24.
- One place to test the relationship (against the real database), small pure
  policies to unit test.
- If a policy ever genuinely needs to know *why* two people can see each
  other, reachability can return reasons again; that is a small change.

## Open questions (to settle here before the authorisation issue)

- The concrete capability list and names.
- **The direct-message policies**, expressed as user stories, not as
  organisation-chart relationships (there is no chart). Stories to satisfy:
  - store staff may not message each other freely, yet may message the
    head-office support desk they share a channel with;
  - some people may message anyone they can see, some nobody;
  - an organisation may allow a named exception ("A may message B").

  Two things are fixed already. First, the shape of the decision:

  ```text
  canStartDM(sender, target):
      if not canReach(sender, target):            return false
      if namedException(sender, target):          return true
      if sender holds dm_reachable:               return true
      if target holds receive_dm_from_reachable:  return true   # candidate
      return false
  ```

  The receiver-side capability is what lets the first story work: the
  head-office support desk holds it, so store staff who share a channel
  with the desk can write to it without holding `dm_reachable` themselves
  and without a named exception per pair.

  Second, **per-user DM exceptions never create reachability**; they only
  relax the rule for a pair that is already reachable. Visibility is
  decided by `canReach` alone.

  Deliberately **not supported**: a user who holds `view_directory` but may
  DM only people they share a channel with. Supporting it would need the
  policy to ask *why* two people are reachable, which is the reason set
  this record removed. For ordinary staff without `view_directory` the
  reachable set already *is* their shared-channel colleagues, so
  channel-level separation holds without a `dm_shared_channel` capability
  or a `sharesChannel` primitive. Starting a DM, posting to an existing
  DM, and adding a participant to a group DM are separate decisions; the
  latter two remain open. Until then invariant I-6 is a TODO.
- What happens to an existing DM when reachability is later lost (proposal:
  history stays, new posts are blocked).
