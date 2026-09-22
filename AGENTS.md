# AGENTS.md

For AI coding tools and the people using them. Short on purpose: it points at
where the rules are, and repeats only the few that are dangerous to miss.

## Read before changing anything

| | |
|---|---|
| [`docs/architecture/invariants.md`](docs/architecture/invariants.md) | Rules that must always hold, and how each is enforced |
| [`docs/domain/glossary.md`](docs/domain/glossary.md) | What words mean. Use these identifiers |
| [`docs/adr/`](docs/adr/) | Why things are the way they are. Do not re-decide a recorded decision inside a pull request; open an issue |
| [`docs/process/workflow.md`](docs/process/workflow.md) | Who does what, and when to stop and ask |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Branches, sign-off, what a pull request needs |

## Never — rules a mechanism enforces

Each of these corresponds to an invariant and is (or will be) backed by a
lint, a source test, a CI job or a database grant. A rule here without a
mechanism yet is an open issue, not a reason to skip the rule.

- **Never let tenant-owned data cross the organisation boundary.** Every
  SELECT, UPDATE and DELETE on an organisation-owned table is scoped by
  `organization_id`; every INSERT stores the correct `organization_id`
  explicitly. No exceptions: channels and everything under them included.
  Channel content additionally requires membership of that channel.
- **Never let an actor span organisations.** Every actor has exactly one
  `organization_id`, set at creation and never changed; the organisation an
  operation runs in is derived from the actor (`token → actor →
  organization`), never taken from the request.
- **Never decide access by role name.** Ask for a capability.
- **Never compute visibility anywhere but in `canReach`.** Two users see each
  other if they share a channel or the viewer holds `view_directory`; no
  other capability grants visibility. There is no organisation hierarchy;
  do not invent one.
- **Never let reachability alone permit a direct message.** DMs have their
  own policies (still being settled in ADR 0005) that combine reachability
  with both people's capabilities and organisation settings.
- **Never grant channel rights outside the channel.** Channel-level rights
  are one flag on the membership, `is_admin`, and mean nothing for any other
  channel.
- **Never point authorship, channel membership or audit at `user`.** They
  reference `actor`; a user is one kind of actor, a bot is another. A bot
  has no directory visibility, is outside reachability, and enters a channel
  only through `canInstallBot`.
- **Never register an HTTP route outside the route table**, and every route
  declares its allowed authentication modes (one or more of `session`,
  `token`, `webhook`; or `public` alone, never combined). The front end
  calls only the versioned public API; there are no front-end-only
  endpoints.
- **Never change a message's id when it moves between topics**, and never
  move a topic to a channel of a different organisation.
- **Never `UPDATE` or `DELETE` an audit record**, not even to fix a mistake.
  A correction is a new record that refers to the wrong one.
- **Never log a message body, password hash, session token, API token,
  invite or login link, email address or username.** Not in errors either.
  The only identifier for a person or bot that may appear in a log is the
  internal `actor_id`.
- **Never write application SQL outside the designated database package**;
  migrations are the exception.
- **Never edit generated code by hand.** Code produced by `sqlc`,
  `oapi-codegen`, the TypeScript client generator or any other tool is
  changed by editing its source (the SQL query file, the OpenAPI document,
  the generator configuration) and regenerating. CI regenerates and fails on
  any difference.
- **Never handle an expected failure with `panic`** — a bad request, a
  database or network error, an external service being down, invalid input.
  `panic` is for bugs (a branch that cannot be reached, a value that cannot
  be nil) and for start-up preconditions (`regexp.MustCompile`,
  `template.Must`, a missing required setting), where failing to start is
  correct. A panic inside a request handler is recovered, logged with the
  request id and answered with 500; it is a bug to fix, not a control flow.
  Tests may assert that something panics. Never call `os.Exit` outside
  `main`. Never ignore an error with `_` without a comment saying why.
- **Never weaken a test or a lint to make it pass.** Deleting a test, loosening
  an assertion, `t.Skip`, `//nolint`, a longer timeout, a golden file updated
  without explanation, or a relaxed linter configuration all need a reason in
  the pull request and an issue that approved it. A red check means the code
  is wrong until shown otherwise.

## Working rules — kept by review and the workflow

- **Never add a dependency without saying why** in the pull request — what
  could not be done without it, and what was considered instead. This
  applies to Go modules and npm packages alike.
- **Never copy code from other chat projects.** Learning from their design is
  fine; pasting their code is not.
- **Do what the issue asks, and no more.** Code outside it is not reformatted,
  renamed or restructured in the same pull request. Something worth fixing
  becomes its own issue.
- **A behaviour change and a refactoring are separate pull requests.**
- **No interface, generic parameter or layer with a single implementation**
  for a need no issue has. A test double counts as an implementation. Share
  code when two places change for the same reason, not because they look alike.
- **Stop and ask** when the plan turns out to be wrong: a schema change the
  plan did not mention, a new dependency, a security finding, a test that
  cannot be written, or a third round of review fixes. The full list is in
  [`docs/process/workflow.md`](docs/process/workflow.md).

## Before saying something works

```bash
make check
```

which runs `gofmt`, `go vet`, `golangci-lint`, `govulncheck` and `go test
./...` (and the front-end equivalents once `frontend/` exists). If `make check`
does not exist yet, run those directly.

**A test for a behaviour change must fail against the code without the
change.** Run it that way once and record the result in the pull request. For
a brand-new capability where the old code does not even compile, say so
instead of pretending a compile error is evidence.

**A concurrency test must run genuinely concurrently.** Sequential execution
cannot exercise the interleaving it exists to catch.

## Tests

- Anything that touches the database is tested against a **real PostgreSQL**
  (testcontainers locally, a service container in CI). No mocks of the
  database.
- Fakes are for **external systems only**: email, push notifications, OIDC
  providers, and similar. Whether object storage is a fake or a real MinIO
  container is decided in the attachments issue.
- Reachability is tested in two layers. **Finding the facts** — does this
  user hold `view_directory`, do these two users share a channel — depends
  on joins and tenant scoping, and is tested against the real database.
  **Deciding** — given those facts, the capabilities involved and the
  organisation's policy, does an action policy hold — is pure and is unit
  tested without a database. A SQL or scoping mistake must not be able to
  hide behind a unit test.
- Other pure logic (validation, formatting, policies) is unit tested and
  covers the success, failure and boundary cases named in the issue's
  acceptance criteria.
- Every HTTP route is registered in one table with its authentication modes
  and scope, and a table-driven sweep applies the negative cases **those
  call for**: a `session` route is called unauthenticated, from the wrong
  organisation, and — if it is channel-scoped — as a non-member; a `token`
  route with a missing, invalid and insufficiently scoped token; a `webhook`
  route with a missing and an invalid webhook credential; a `public` route
  gets no authentication-rejection case. A route that is not in the table
  does not exist.

## Front end

Each concern has one tool. Use the one for the layer you are in.

| Layer | Use | Not |
|---|---|---|
| Structure | Semantic HTML and browser-standard elements first: `<dialog>`, `<details>`, `<form>` with native validation, `<select>`, `<button>` | `<div>` with click handlers |
| Appearance | Tailwind with the shared theme (colours, spacing, type). Components normally use Tailwind rather than inventing their own CSS. Arbitrary values (`w-[13px]`, `text-[#123456]`) need a reason in the pull request, as does hand-written CSS outside the narrow exceptions: the central theme and global styles, CSS custom properties, and integrations where Tailwind does not fit | a private CSS system per component, inline styles |
| State and interaction | The least Svelte 5 / TypeScript that does it. Component-local UI state (a menu open or closed, the selected tab, an input draft, focus) lives in the `.svelte` file. Reusable or non-trivial front-end-only logic lives in `.ts` modules with Vitest tests | product decisions in the front end |
| Complex accessible widgets (menu, combobox, dialog with focus trap) | shadcn-svelte components (copied into the repository, ours to edit) over Bits UI | a hand-rolled widget |
| Business rules, authorisation, data decisions | Go. **The UI hides; the server denies.** The front end may hide a control the API says the actor cannot use, and may validate for the user's convenience, but it never decides | any check on the client that the server does not also make |

- Runes only: `$state`, `$derived`, `$props`, `$effect`. Legacy component
  syntax (`export let`, `$:`, `on:click`, stores for component state) is
  forbidden in this repository: `compilerOptions.runes = true` makes the
  compiler reject it, and CI must reject it. Do not work around it.
- `$derived` for values computed from state; `$effect` only for side effects
  that leave the component (a WebSocket subscription, focus, a timer).
- No SvelteKit server code: no `+page.server.ts`, `+server.ts`, form actions
  or `load` functions that talk to a database. The Go server is the only
  back end; call it through the generated API client.
- When unsure of Svelte 5 behaviour, consult the Svelte MCP server when
  available, otherwise the official Svelte documentation. Do not rely on
  memory. `svelte-check` must pass.

## Writing

- Code, comments, documentation, commits, issues and pull requests are
  **English**.
- **Comments say why, not what.** A rejected alternative, a trap, the reason an
  order matters, something deliberately not done.
- Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/)
  and carry a `Signed-off-by` line (`git commit -s`).
- Work on a branch named `issue-<number>-<short-slug>`, never on `main`.
