# 0002. Go server with an embedded Svelte single-page app

Date: 2026-09-22. Status: accepted.

## Context

The maintainer's interest is overwhelmingly the back end, and the project is
a vehicle for learning Go. A chat product without a usable front end is not a
portfolio piece, and users judge a chat app by its front end. The front end
will therefore be written mostly by AI tools, and reviewed rather than
hand-written — and the maintainer wants to be able to *read* it.

Options considered for the client: a server-rendered UI embedded in Go
(templ + htmx), a separate single-page app, or API-only for now. For the SPA
framework: React, Svelte 5 and Solid.

React has by far the largest training corpus, so every AI tool produces and
reviews it most reliably, and Mattermost and Rocket.Chat are React. Svelte 5
needs markedly less code and is easier to read, and updates the DOM at signal
granularity. Neither performance argument is decisive for a chat client,
where perceived speed comes from list virtualisation and batching of
WebSocket updates. Solid's ecosystem is too small. The decisive trade is
**reliability of AI output (React) against readability for the maintainer
(Svelte)**; the maintainer chose readability, accepting the risks below and
the mitigations that go with them.

## Decision

- A single-page app from the start, in **Svelte 5 + TypeScript**, built with
  **SvelteKit in static SPA mode** (`adapter-static`, `ssr = false`) for its
  routing and layouts. It lives in `frontend/` in this repository.
- The production build is embedded in the Go binary with `embed`, so one
  binary and one container image serve both API and UI.
- The contract between the two is the **OpenAPI document** (ADR 0012); the
  TypeScript client is generated from it.

## One tool per layer

To keep AI output consistent, each front-end concern has exactly one tool:

| Layer | Tool |
|---|---|
| Structure | semantic HTML, browser-standard elements first |
| Appearance | Tailwind with a shared theme; hand-written CSS only for the theme, global styles, custom properties and integrations where Tailwind does not fit, with a reason |
| State and interaction | the least Svelte 5 / TypeScript that does it; component-local UI state stays in the component, reusable or non-trivial logic in `.ts` modules with unit tests |
| Complex accessible widgets | shadcn-svelte (copied in, ours to edit) over Bits UI |
| Business rules, authorisation, data decisions | Go. The UI hides; the server denies |

## Risks of writing Svelte with AI tools, and the mitigations

| Risk | Mitigation |
|---|---|
| Svelte 4 and 5 syntax mixed in one file (`export let` vs `$props()`, `$:` vs `$derived`, `on:click` vs `onclick`, stores vs `$state`) | Legacy component syntax is forbidden in this repository: `compilerOptions.runes = true` in `svelte.config.js` makes the compiler reject it, and `svelte-check` plus `eslint-plugin-svelte` in CI reject anything that slips through |
| Inconsistent styling across generated components | Tailwind restricted to theme tokens; arbitrary values need a reason |
| Subtle reactivity mistakes from a smaller training corpus (`$effect` used to compute derived values, mutation of non-state objects) | The official Svelte MCP server and Svelte's `llms.txt` documentation are connected to the tools that write and review front-end code; a short front-end section in `AGENTS.md`; Vitest + Testing Library and a few Playwright flows |
| Reviewer tools (Copilot, Codex) are also weaker at Svelte | Front-end correctness rests on `svelte-check` and tests, not on review. Authorisation and tenant logic live on the server, so front-end pull requests are `risk:low` or `risk:medium` |
| Smaller ecosystem; tools misremember library APIs | Few dependencies, each justified in its pull request. Framework-agnostic choices where possible: `openapi-typescript` + `openapi-fetch`; TanStack Query and Virtual via their Svelte adapters only if plain modules prove insufficient; shadcn-svelte / Bits UI for complex widgets |
| Tools reach for SvelteKit server features (`+page.server.ts`, form actions) that do not apply with a Go back end | SPA mode makes them build errors; a source test forbids `*.server.ts` files under `frontend/` |

## Consequences

- The repository is a monorepo with a Go root and a `frontend/` package.
- CI runs both toolchains, including `svelte-check`. The Docker build is
  multi-stage.
- The maintainer reviews front-end code rather than writing it; the OpenAPI
  document is the place to be careful.
- This record replaces the React decision taken earlier the same day, before
  anything was pushed; no code was affected.
