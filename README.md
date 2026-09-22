# go-go-chat

A self-hostable team chat server for organisations with a hierarchy — head
office, regions, stores — where who can talk to whom is a policy, not an
accident. Written in Go, with a React front end embedded in a single binary.

**Status: pre-alpha. Nothing runs yet.** The repository currently holds the
design decisions and the development process; code arrives issue by issue.

## What it will be

- **Channels and topics, Zulip style.** A channel is a stream; a topic is a
  conversation inside it. The default view of a channel is a single
  chronological feed of every topic, so replies never disappear into the past.
  Posting without a topic is fine — replying to such a message *creates* the
  topic. Anyone with the capability can move messages into a better topic
  later.
- **A group tree per organisation.** Organisation › area › store, or any shape
  up to a fixed maximum depth (three levels in v1). Channels belong to a group. Who can see and
  message whom is derived from the tree and from organisation policy, so a
  part-timer in one store never sees the staff of another unless the
  organisation says so.
- **Shared channels.** A channel can span several groups (and, later, several
  organisations) without loosening the boundary for everything else.
- **One API for everyone.** The web app is an ordinary client of the
  versioned public API, so a third-party integration or an incoming webhook
  can do whatever the app can, with its own token and its own audit trail.
- **One binary, one `compose.yaml`.** PostgreSQL plus the server. Download and
  run.

Longer term: Valkey-backed horizontal scaling, load testing, a WebAssembly
plugin sandbox, OIDC. See the [architecture decision records](docs/adr/) for
what has been decided and why.

## Why this exists

This is a hobby project by one person, built to learn Go properly and to
practise a production-like development process with AI tooling in the loop.
It is public from the first commit so the process can be seen, not just the
result. See [docs/process/workflow.md](docs/process/workflow.md).

## Documentation

| | |
|---|---|
| [docs/adr/](docs/adr/) | Architecture decision records — the design, one decision per file |
| [docs/domain/glossary.md](docs/domain/glossary.md) | What words mean. Use these identifiers |
| [docs/architecture/invariants.md](docs/architecture/invariants.md) | Rules that must always hold, and how each is enforced |
| [docs/process/workflow.md](docs/process/workflow.md) | How an issue becomes a merged pull request |
| [AGENTS.md](AGENTS.md) | Rules for AI coding tools and the people using them |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Branches, sign-off, what a pull request needs |

## License

[MIT](LICENSE).
