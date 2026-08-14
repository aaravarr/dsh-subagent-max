# Agent Note: Single package, dual entry points

Status: implemented

## Problem

The plugin has two faces with different load paths — a Node/Cordis host entry and a browser client entry — and both must be shipped, versioned, and published together without forcing consumers to install two things.

## Decision

The project ships as one npm package, `@aaravarr/dsh-subagent-max`, with two entry points in `package.json` `exports`: `"."` → `./lib/index.js` (host) and `"./client"` → `./lib/client.js` (client), plus `"./package.json"` for tooling. `files` is `["lib"]`, and the DSH client metadata declares `dsh.client.platform: "web"` with the `@deepseek-ai/dsh-client-ui-primitives` injection so the client entry loads only in web profiles. `main` points at `lib/index.js`.

One package with two entries is chosen: both faces share the same identity (`name` and `id`), the same subagent session model, and one version/release line, while each entry keeps its own runtime and load path (Cordis `apply` for the host, `__ModuleLoader__.load` for the client).

## Alternatives considered

- **Two separate packages** (host-only and client-only) — rejected: the faces are tightly coupled to the same subagent model and the same `id`/`name`; splitting doubles release, version, and i18n sync overhead for no runtime benefit.
- **Folding the client into the host entry** — rejected: the host runs in Node under Cordis and the client runs in the browser under `__ModuleLoader__`, so they cannot share a single load path; merging would force bundling a web payload into the host.
- **A build/bundler step to emit two artifacts** — rejected: the project is plain ESM with no build step (`node --check` only), so two entry files under `exports` are simpler than introducing a bundler.

## Consequences

- One install gives consumers both faces; one `npm version` + `git push --follow-tags` publishes both.
- `dsh.client.platform: "web"` constrains the client to web profiles while the host face stays platform-agnostic.
- `__ModuleLoader__.load({ id })` and `package.json` `name` must stay equal — a rename touches both, which `AGENTS.md` calls out as a hard constraint.
