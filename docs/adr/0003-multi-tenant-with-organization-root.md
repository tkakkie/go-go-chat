# 0003. Multi-tenant data model with the organisation as root

Date: 2026-09-22. Status: accepted.

## Context

A self-hosted installation typically serves one company. Making the
installation itself the tenant is simpler now, but adding a tenant column to
every table later is surgery on the whole schema. Keeping a tenant boundary
from the first migration costs little and teaches the single most important
discipline in this kind of system.

## Decision

- An `organization` table is the root. Every organisation-owned row carries
  `organization_id`, and every query against such a row is scoped by it.
- The v1 user interface and setup create exactly one organisation. Multiple
  organisations per installation are a data-model property, not a v1 feature.

## Consequences

- Every issue that touches the schema is `risk:high` until the scoping is
  enforced by a mechanism (invariant I-1).
- The organisation is an absolute boundary in v1; sharing a channel with
  another organisation, if it ever comes, needs its own ADR (see ADR 0007).
