# 0009. AI-assisted development workflow with human gates and provenance

Date: 2026-09-22. Status: accepted.

## Context

The project exists as much to practise a production-like process with AI
tools as to produce a chat server. The maintainer wants to delegate as much as
possible to Claude, to have every AI-produced artefact reviewed by a
*different* AI, to keep human review light by making tests carry the weight,
and to route heavier review to the changes that deserve it.

The first draft had eleven hand-offs per issue. Review of that draft, and of
this record's predecessors, produced the shape below.

## Decision

The full process is in [`docs/process/workflow.md`](../process/workflow.md).
The decisions it rests on:

- **Different-AI review.** Claude writes requirements and plans; Codex
  reviews them. Codex writes code; Claude reviews it, Copilot reviews
  automatically, Jules reviews adversarially for `risk:high`.
- **Three human gates**: (1) issue approval; (2) plan approval; (3) pull
  request approval and merge. Between the second and third, Claude
  orchestrates without asking unless a listed stop condition applies.
- **Risk labels route review.** `risk:high` (auth, authorisation,
  migrations, sessions, upload, tenant boundary) gets Jules; the rest does
  not. Jules is set up before the first `risk:high` issue; Codex and Copilot
  are used from the first issue.
- **Provenance over authorship.** Codex runs as a local CLI driven by Claude
  so the fix loop has one orchestrator; the GitHub author field is
  incidental. Every pull request records who produced and reviewed what.
- **Issues split by behaviour, not by line count.** A `size:*` label is a
  signal of review effort only.
- **Tests carry the weight.** New tests are shown failing on the base commit
  (or the pull request explains why they cannot compile there); CI is strict;
  reviewers focus on acceptance criteria, invariants and scope, and still
  report any concrete correctness or security issue they discover.
- **Evidence over speculation in AI review.** A question that a small
  experiment can answer safely is answered by the experiment during
  implementation, not by prolonged discussion at issue or plan review.
- Conventions: ADRs, Conventional Commits with release-please, GHCR images on
  tags, DCO, branch protection, gitleaks, CodeQL, govulncheck, Dependabot,
  and a versioned `/ship` routine under `.claude/`.

## Consequences

- Process tooling (labels, templates, hooks, the `/ship` routine, Jules
  action) is real work with its own issues.
- The workflow is expected to change as it is used; changes are recorded
  here as superseding records, not made silently.
