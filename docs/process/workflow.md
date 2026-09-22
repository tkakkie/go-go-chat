# Development workflow

How an issue becomes a merged pull request, and who — person or tool — does
what. This is the process the project exists to practise, so it is written
down in full.

## Roles

The rule that shapes everything: **whatever an AI produces is reviewed by a
different AI**, and a human approves at fixed gates.

| Artefact | Produced by | Reviewed by |
|---|---|---|
| Requirements (the issue) | Claude | Codex, then a human approves |
| Implementation plan (similar code, test approach, split) | Claude | Codex, then a human approves |
| Code and tests | Codex CLI, driven by Claude in a worktree | Claude (always), Copilot (automatic), Jules (`risk:high` only, adversarial) |
| Pull request text, triage of review comments | Claude | A human checks and merges |

Claude never reviews its own plan; Codex never reviews its own code.

Codex runs as a local CLI rather than as a cloud task so that the fix loop
(implement → Claude review → fix) stays under one orchestrator. Which name
appears as the GitHub author is not the point; **provenance is**. Every pull
request records it:

```text
Requirements: Claude
Plan: Claude
Implementation: Codex CLI
Primary review: Claude
Automated review: Copilot
Adversarial review: Jules (risk:high only)
Human approval: required
```

## Human gates

1. **Issue approval.** The issue satisfies the Definition of Ready: a problem
   statement, scope and non-goals, acceptance criteria that can be checked, a
   test plan, and `risk:*` and `type:*` labels.
2. **Plan approval.** The plan names the files it expects to touch, the
   similar code it will reuse or deliberately not reuse, the tests it will
   add, and whether the issue should be split.
3. **Pull request approval and merge.** Merging is done by a human, by squash,
   only after CI is green and every review the risk level requires has
   completed.

Between gates 2 and 3 the orchestrator proceeds without asking, **unless** one
of the stop conditions below applies.

## Stop conditions

Stop and ask a human when:

- the work would change scope, alter the schema in a way the plan did not
  mention, or add a dependency;
- reviewers disagree, or Jules reports a security finding;
- a test for an acceptance criterion cannot be written;
- the implement–review–fix loop exceeds three rounds.

## Risk levels

| Label | Applies to | Reviews required | What the human reads |
|---|---|---|---|
| `risk:high` | authentication, authorisation and reachability, migrations, sessions, file upload, tenant boundary | Claude + Copilot + Jules | the whole diff |
| `risk:medium` | API changes, realtime, data model | Claude + Copilot | the parts the pull request marks "read this" (chosen by the reviewing AI, and worth reading to learn from), plus the summary, CI and review results |
| `risk:low` | docs, cosmetics, renames | Claude + Copilot | the summary |

**Refactoring is classified by the area it touches, not by the claim that
behaviour does not change.** A large refactor of the authorisation code is
`risk:high`; a restructuring of the realtime internals is `risk:medium`;
tidying variable names is `risk:low`. When a change touches several areas,
the highest applies. A refactor is still its own pull request, separate from
behaviour changes.

Jules is configured before the first `risk:high` issue (authentication); it is
not needed for the repository skeleton, CI and database plumbing that come
first.

## Splitting work

Issues are split by **behaviour**, not by line count, and the decision is
made at the gate where the information exists:

| When | Split if | Otherwise |
|---|---|---|
| Issue approval | the issue plainly has several reasons to change | approve as one |
| Plan approval | the plan reveals that the implementation is too large to review as one change, or mixes risk levels | approve as one |
| After implementation | never split after the fact when the scope is still one behaviour and the diff is large only because of generated code or tests | the pull request tells the reviewer where to read first, what is generated, and what needs judgement |
| During implementation | the scope itself has grown | **stop condition** — a human decides |

A `size:*` label is added automatically from the diff (excluding generated
code, lock files, migrations and tests) as a signal of review effort, nothing
more.

## Tests carry the weight

Reviews stay light because CI is strict:

- Go unit tests; integration tests against a real PostgreSQL (service
  container in CI, testcontainers locally); contract tests generated from the
  OpenAPI document; `svelte-check`, Vitest and a small number of Playwright
  flows for the front end; k6 load tests in a separate, manually triggered
  workflow.
- A pull request whose CI is red is not reviewed.
- **A test for a behaviour change is shown failing before the change.** The
  `/ship` routine runs the new tests against the base commit and pastes the
  result into the pull request. For a new capability whose tests cannot
  compile against the base, the pull request says so instead.
- **Coverage is information, not a gate.** A percentage invites tests that
  raise the number without protecting anything. The pull request instead
  states which behaviour changed, which boundary conditions matter, and
  what must not break together with the test that proves it; diff coverage
  is posted as a comment so a reviewer can ask why a changed line has no
  test.
- Reviewers therefore focus on three things: does it meet the acceptance
  criteria, does it respect [`invariants.md`](../architecture/invariants.md),
  and does it change anything outside the issue. A concrete correctness or
  security problem noticed along the way is still reported; the focus
  limits scope creep, not honesty.

## Evidence over speculation

AI reviewers, and the humans reading them, prefer a measurement to a guess.

- A question that a small implementation, test, benchmark or throwaway
  script can answer safely and cheaply is **not** debated at length in issue
  or plan review. The plan records the open point and how it will be
  verified; implementation verifies it.
- Before repeating a fact-finding investigation, ask first: **can a test
  settle this?** If yes, write the test.
- What a local experiment cannot settle — an official guarantee, a security
  property, a compatibility promise, a licence — is researched in
  documentation and other external sources, and the source is cited.

## Conventions that make this feel like a team

- Decisions are recorded as ADRs in [`docs/adr/`](../adr/).
- Commits follow Conventional Commits; release-please generates the changelog
  and a tag publishes a container image to GHCR.
- Branch protection on `main`, DCO check, `gitleaks` in the pre-commit hook and
  in CI, CodeQL, `govulncheck`, Dependabot.
- The `/ship <issue>` routine that drives gates 2 → 3 lives in this repository
  under `.claude/`, so the process is versioned with the code.
