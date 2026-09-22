# go-go-chat

A self-hostable team chat server for organisations with many locations —
head office, regions, stores — where who can talk to whom is a policy, not
an accident. Written in Go, with a Svelte front end embedded in a single binary.

**Status: pre-alpha. Nothing runs yet.** The repository currently holds the
design decisions and the development process; code arrives issue by issue.

## What it will be

- **Channels and topics, Zulip style.** A channel is a stream; a topic is a
  conversation inside it. The default view of a channel is a single
  chronological feed of every topic, so replies never disappear into the past.
  You can pick or create a topic when you post, or post without one —
  replying to such a message *creates* the topic by default. Anyone with
  the capability can move messages into a better topic later.
- **Isolation without an org chart.** A store is a private channel. People
  who do not share a channel do not see each other, unless they hold the
  capability to see the whole directory. Who may start a direct message,
  and who may create channels, is organisation policy. No hierarchy to
  model, nothing to get wrong.
- **One API for everyone.** The web app is an ordinary client of the
  versioned public API. A third-party integration uses the same API, subject
  to what its actor is allowed to do and to its token's scope, with its own
  audit trail. Incoming webhooks post through a bot with a per-webhook secret.
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
