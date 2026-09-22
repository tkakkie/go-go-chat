# 0001. Record architecture decisions

Date: 2026-09-22. Status: accepted.

## Context

This project is built by one person with several AI tools. Decisions made in
conversation are lost unless written down, and a tool asked to implement an
issue will happily re-decide something that was settled last week.

## Decision

Keep architecture decision records in `docs/adr/`, one per decision, in the
format used here. Before a record is first merged to `main` it may be
edited freely. Once on `main`, a record is never deleted and its decision is
not materially changed in place; it is superseded by a later record.
A pull request that makes a decision future work must respect includes a
record. A pull request must not reverse a recorded decision; that is an issue
and a new record.

## Consequences

- Slightly more writing per change.
- Tools and people can be told "read the ADRs" instead of being told the
  design again.
