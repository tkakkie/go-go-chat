# Architecture decision records

One decision per file, numbered, never deleted. A superseded record stays and
points at its successor. Format: context, decision, consequences, open
questions with the issue where each will be settled.

The first twelve records were written at kick-off on 2026-09-22 from a design
discussion between the maintainer and Claude, and revised on 2026-09-23
through several review rounds with ChatGPT, including the move from a group
tree to a flat organisation (0004). Later records arrive with the pull
requests that need them.

| # | Title |
|---|---|
| [0001](0001-record-architecture-decisions.md) | Record architecture decisions |
| [0002](0002-go-server-with-embedded-svelte-spa.md) | Go server with an embedded Svelte single-page app |
| [0003](0003-multi-tenant-with-organization-root.md) | Multi-tenant data model with the organisation as root |
| [0004](0004-flat-organisation.md) | A flat organisation: no hierarchy in authorisation |
| [0005](0005-reachability-and-action-policies.md) | Reachability as a primitive, authorisation as per-action policies |
| [0006](0006-channels-topics-and-the-feed.md) | Channels and topics, with a feed-first view and reply-creates-topic |
| [0007](0007-channels.md) | Channels: public or private, explicit members, one organisation |
| [0008](0008-username-password-authentication.md) | Username and password authentication, credentials separated from users |
| [0009](0009-ai-assisted-development-workflow.md) | AI-assisted development workflow with human gates and provenance |
| [0010](0010-defer-rust-and-webassembly.md) | Defer Rust and WebAssembly to later milestones |
| [0011](0011-initial-technology-choices.md) | Initial technology choices |
| [0012](0012-public-api-and-integrations-from-the-start.md) | Public API and integrations from the start |
