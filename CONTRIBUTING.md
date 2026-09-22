# Contributing

Thanks for looking. This is a hobby project run by one person, built in public
to practise a production-like process. A few things are stricter than you
might expect — they exist to keep it maintainable by one person and safe to
run in public.

## Before writing code

**Open an issue first.** A pull request that arrives without one is likely to
conflict with something already decided but not yet written down. Issues use
the templates in `.github/ISSUE_TEMPLATE/`; the important parts are the
acceptance criteria, the test plan and the `risk:*` label.

Read [`docs/process/workflow.md`](docs/process/workflow.md) for how an issue
travels to a merged pull request, and [`AGENTS.md`](AGENTS.md) for the rules
that apply to every change whether a person or a tool writes it.

## Sign your commits off (DCO)

Every commit must carry a `Signed-off-by` line certifying the
[Developer Certificate of Origin](https://developercertificate.org/):

```bash
git commit -s -m "feat: your message"
```

The sign-off states that you wrote the change or otherwise have the right to
submit it under this project's licence. There is no separate contributor
agreement.

## Ground rules

- **Work on a branch, never on `main`.** Run
  `git config core.hooksPath .githooks` after cloning; the hooks refuse commits
  and pushes on `main`, and scan staged changes for secrets with `gitleaks`.
- **Branch names** are `issue-<number>-<short-slug>`.
- **Commit messages** follow Conventional Commits (`feat:`, `fix:`, `docs:`,
  `refactor:`, `test:`, `chore:`, `ci:`). Release notes are generated from
  them.
- **English** in code, comments, documentation, commit messages, issues and
  pull requests.
- **Do not add a dependency without saying why** in the pull request.
- **Do not copy code from other chat projects.** Learning from their design
  is fine; pasting their code is not.
- **`panic` is not error handling.** Expected failures — bad input, database
  or network errors, an external service being down — are returned as
  errors. `panic` is for bugs and start-up preconditions only; see
  [`AGENTS.md`](AGENTS.md). No `os.Exit` outside `main`.
- **Errors are never silently ignored.** Discarding an error with `_` needs a
  comment saying why that is correct there.

## What a pull request needs

- A linked issue, and a plan that was approved on that issue.
- The **provenance block** from the pull request template filled in: who wrote
  the requirements, the plan, the implementation, and who reviewed it.
- A description of what changes and why, and where a reviewer should look
  first if the diff is large.
- Evidence that the tests test the change. For a bug fix or a behaviour
  change, run the new tests against the base commit, **check that they fail
  for the intended reason**, and record the result in the pull request. For
  a new capability whose tests cannot compile against the base, say so
  instead; a compile error is not the evidence being asked for. A test that
  passes against the unchanged code is not evidence of anything.
- `make check` passing (or, before the Makefile exists: `gofmt -l .` empty,
  `go vet ./...`, `golangci-lint run`, `govulncheck ./...`, `go test ./...`).
- If the change adds or relies on an invariant, an entry in
  [`docs/architecture/invariants.md`](docs/architecture/invariants.md) with
  its enforcement mechanism. If enforcement still depends on a reviewer
  noticing, say so in the *Enforced by* cell and cite the issue that will
  replace it with a mechanism.
- If the change makes a decision that future work must respect, an ADR in
  [`docs/adr/`](docs/adr/).

Pull requests are merged by a human, with squash, after CI is green and the
reviews listed for the issue's risk level have completed.
