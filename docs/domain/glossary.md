# Glossary

Use these identifiers in code, schema, API and documentation. When a word here
does not fit, propose a change to this file in the same pull request rather
than inventing a synonym. This is a dictionary — what a word means and how it
differs from its neighbours — not a specification; algorithms, schema and API
contracts live in the ADRs, the invariants and the issues.

## Tenancy and people

| Term | Identifier | Meaning |
|---|---|---|
| Organisation | `organization` | The tenant, and the absolute boundary: nothing is shared across organisations. The root that owns everything. One installation can hold many; v1 UI creates one. There is no hierarchy inside it (ADR 0004). |
| Role | `role` | A named bundle of capabilities, defined **per organisation** (one organisation may have `manager` and `part_time`, another `supervisor` and `staff`), granted to a user for the whole organisation. A user may hold several. Never used directly for an access decision. |
| Capability | `capability` | A single **organisation-wide** permission (create a channel, invite to a channel, see the directory, ...). The unit of organisation-level authorisation. Channel-local authorisation is deliberately limited to the `is_admin` flag on a channel membership and is not a capability. A user's effective capabilities are the **union** of all their roles; there is no deny and no priority. `view_directory` is the one capability that affects who a user can see. |
| Directory | — | The list of all users in the organisation. Visible only to users with `view_directory`; everyone else sees only the users they share a channel with. |
| User group | (future) | A flat list of users (not nested) for adding many people to a channel at once. Not yet designed. Will never be an authorisation primitive; it only expands to individual memberships. |

## Actors

| Term | Identifier | Meaning |
|---|---|---|
| Actor | `actor` | The common principal: whoever authors a message, holds a channel membership, owns an API token, or appears in an audit record. Either a user or a bot. Belongs to exactly one organisation, fixed at creation. Roles and reachability apply to users only. |
| User | `user` | A human actor. Signs in with a `username` unique within the organisation and never containing `@`; email is optional. Holds roles. |
| Bot | `bot` | A non-human actor owned by an organisation for integrations and incoming webhooks. Authenticates to the general API only with API tokens; an incoming webhook authenticates its own secret and then acts as its bound bot. May be a channel member; has no directory visibility; outside reachability and direct messages. Holds no roles; what a bot may do is fixed and narrow (ADR 0012). |
| Credential | `credential` | A way an actor proves identity: password now, API token and OIDC later. Separate from the actor. |
| API token | `api_token` | A credential for calling the API without a browser. Belongs to one actor, so to one organisation. Its scope can only **narrow** the actions its actor is otherwise allowed to perform — a user's from roles and capabilities, a bot's from its fixed bot permissions — and never grants a new one. Stored hashed; never logged. |
| Session | `session` | A server-side login session referenced by an HttpOnly cookie. |

## Reaching people

| Term | Identifier | Meaning |
|---|---|---|
| Reachability | `canReach` | Whether user A can see user B: they share a channel, or A holds `view_directory`. Directional. Bots are outside it. Computed in one place; an input to action policies, never itself a permission. |
| Action policy | — | A per-action authorisation decision (`canDM`, `canInviteToChannel`, `canInstallBot`, ...). Depending on the action it may consider reachability, capabilities, channel membership and organisation settings; `canInstallBot`, for example, never consults reachability. Direct-message policies are still being decided in ADR 0005. |
| Direct message | `dm` | A private conversation between two **users**, or among several (group DM). Bots do not take part. Who may start one, post to one, or be added to one is decided in ADR 0005, not here. |

## Channels and conversation

| Term | Identifier | Meaning |
|---|---|---|
| Channel | `channel` | A stream of conversation belonging to one organisation. `public` (any user of the organisation can discover and join) or `private` (members only; membership by invitation or at onboarding). A store is typically a private channel; announcements are public. |
| Channel membership | `channel_member` | An **actor's** actual participation in a channel. A user is a member of `#shinjuku` and perhaps not of `#recruiting`; a bot can be a member of `#development`. Obtained by invitation, by self-join on a public channel, at onboarding of a new user, or by bot installation. Carries one flag, `is_admin`, the only channel-local authorisation. |
| Topic | `topic` | A conversation inside a channel. Zulip's "topic"; the unit that behaves like a thread. Every message is in exactly one topic. |
| Default topic | (name TBD) | The topic a message lands in when none is given. **Not** named the same as the default channel. |
| Feed | — | The default view of a channel: every topic's messages interleaved chronologically, each labelled with its topic. |
| Message | `message` | A post in a topic by an actor. Its id never changes, including when it is moved to another topic. |
| Move | — | Reassigning messages (or a whole topic) to another topic or channel. A domain event. Cross-organisation moves are not allowed. |

## Integration and record

| Term | Identifier | Meaning |
|---|---|---|
| Webhook (incoming) | `webhook` | A per-webhook secret bound to a bot and a channel. An external system presents the secret; the server authenticates the webhook, not the bot, and the post is made as the bound bot. |
| Domain event | `event` | A named, versioned record of something that happened (message posted, topic moved). Consumed by realtime delivery, unread counts, audit and outgoing webhooks. |
| Authentication modes | — | Which credentials a route accepts: one or more of `session`, `token`, `webhook`; or `public` alone, never combined. Declared in the route table. |
| Audit record | `audit_record` | An append-only record of a security-relevant action, attributed to an actor. Never updated or deleted; a correction is a new record referring to the old one. |
