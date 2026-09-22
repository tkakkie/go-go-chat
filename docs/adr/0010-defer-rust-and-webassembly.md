# 0010. Defer Rust and WebAssembly to later milestones

Date: 2026-09-22. Status: accepted.

## Context

The maintainer wants to try Rust compiled to WebAssembly somewhere in the
project. Putting it in the core would make Rust a hard dependency of the
build and shift the project away from Go.

## Decision

Nothing in v1 uses Rust or WebAssembly. Two milestones are reserved, in this
order:

1. **Client-side image preprocessing** before upload (resize, compress, strip
   EXIF) via wasm-bindgen. Isolated, measurable, establishes the toolchain.
2. **A server-side WebAssembly plugin sandbox** (wazero, likely via Extism)
   for bots, integrations and message filters. The extension point for
   growing towards Mattermost/Zulip-class integrations, and the strongest
   showcase.

Sharing a Rust Markdown/mention parser between browser and server as one
WebAssembly module is attractive but parked: it would make the core depend
on Rust before the toolchain is proven.

Not suitable: end-to-end encryption (conflicts with ADR 0008), rendering or
virtual scrolling (the DOM is JavaScript's job), client-side full-text search
(PostgreSQL does it).

## Consequences

- Rust code, when it arrives, lives in its own directory with its own CI job
  and Docker build stage.
