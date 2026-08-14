# AGENTS.md — dsh-subagent-max

Guidelines for AI agents working in this repository. This is a two-face plugin for [DeepSeek Harness (DSH)](https://github.com/deepseek-ai/DeepSeek-Harness) — follow these rules so changes stay additive and the client bundle stays intact.

## 1. Project overview

- **Host face** (`lib/index.js`) — a Cordis plugin (`name: "dsh-subagent-max"`, `inject: ["tools", "subagents"]`) that registers the `subagent_with_model` tool: a thin wrapper over `ctx.subagents` that forwards per-call `model` / `provider` into the child's `agentOptions`. Config (schemastery): `subagentProvider`, `toolName`, `backgroundMode`, `maxDepth`.
- **Client face** (`lib/client.js`) — a Web UI plugin loaded via `window.__ModuleLoader__.load({ id: "@aaravarr/dsh-subagent-max", factory })`; `inject: ["sessions", "connection", "slots", "locale"]`. It renders floating popups (`shell.overlay`), a **Subagents** tab card grid (`conversation.view`), drag-to-popup, notifications, and zh/en i18n.

## 2. Hard constraints

- **Fully additive** — never modify any `@deepseek-ai/dsh-*` official package; only use DSH's public APIs (`ctx.subagents`, `ctx.tools.register`, `api.events.mux/host`, `ctx.slots.inject`, `ctx.locale`) and `@deepseek-ai/dsh-client-ui-primitives` exports.
- **Never read-then-rewrite `lib/client.js` wholesale** — its CSS is one giant single-line string; a whole-file read + write can inject `line truncated` and corrupt the bundle. Use precise `edit` replacements only.
- **Verify after every `lib/client.js` edit**: `node --check` passes, grep shows no `line truncated`, and the deployed copy under `<profile>/node_modules/@aaravarr/dsh-subagent-max/lib/client.js` hash-matches the source.
- **`__ModuleLoader__.load({ id })` must equal `package.json` `name`** — both are `@aaravarr/dsh-subagent-max`; keep them in sync when renaming.

## 3. Coding conventions

- **Plain JavaScript (ESM), no TypeScript, no JSX** — React uses `React.createElement(...)`.
- **`dsm-` class prefix** for all component styles.
- **DSH tokens only** — use `var(--dsw-*)` with a dark fallback (e.g. `var(--dsw-alias-label-primary, #e6e6e6)`); don't invent tokens.
- **i18n for all user-facing text** — call `T(key)`; add any new key to both the `ZH` and `EN` dictionaries in `lib/client.js`.

## 4. Useful commands

| Command | Purpose |
| --- | --- |
| `node --check lib/index.js lib/client.js` | Syntax-check both entries (there is no build step). |
| `pnpm install` | Install dependencies. |
| `dsh plugin --profile web add @aaravarr/dsh-subagent-max` | Install into a web profile. |

## 5. Release

- `npm version patch` (or `minor` / `major`) bumps the version and creates a `v*` tag.
- `git push --follow-tags` triggers `.github/workflows/publish.yml` → `npm publish --access public` using the `NPM_TOKEN` secret.

## 6. Agent Notes

- **Record non-trivial changes as notes** — any change that alters behaviour, contracts, or architecture adds or updates a note under `.agents/notes/implemented/` (path `topic/yyyy-mm-dd-topic-title.md`, e.g. `architecture/`, `feature/`).
- **Fixed four-section shape** — `# Agent Note: <title>` + `Status: implemented`, then `## Problem` / `## Decision` / `## Alternatives considered` / `## Consequences`.
- **Decision is present tense** — describe what is actually shipped, not what was planned.
- **Alternatives must be real** — name each alternative actually weighed and why it lost.
- **Never rewrite a note into a different decision** — if a decision is superseded, write a new note and cross-link both; leave the old one intact.

## 7. Writing standards

- **Write full contracts** — for every actor or system, state the actors, ordering/timing, modality (may / must / will not), negative guarantees, and side effects; leave none implicit.
- **Trim reasoning, repetition, decoration** — delete chain-of-thought transcripts, repeated statements, and ornamental filler; keep only what the next reader needs.
- **Pass the CoT-leak test** — a HEAD reader with no session or PR thread must resolve every reference; if a sentence only makes sense with context that isn't in the file, rewrite or delete it.
- **No slop** — avoid stacked filler words, emoji with no information, and empty promises ("we will improve robustness"); state what is true and concrete.
- **README is bilingual and mirrored** — update `README.md` and `README.zh.md` together, mirror the structure, and keep terminology consistent across both.

## 8. Code quality

- **Prove consumers before simplifying** — before removing or "simplifying" an API, helper, or export, search call sites and separate production vs non-production consumers; don't delete something just because this repo no longer references it.
- **Run minimal relevant checks before push** — run the `node --check` from Useful commands; if it fails, stop and fix rather than forcing a broken push. Use `git push --force-with-lease`, never bare `--force`.
- **Review both faces** — for this dual-face plugin, verify host/client contract consistency, that schemastery schemas match the runtime shapes, that `ZH`/`EN` i18n keys stay in sync, and that React unmount/effect cleanups (listeners, timers, `AbortController`, `store.dispose`) run.

---

Keep changes additive, verify the client bundle after every edit, and add i18n keys for any new text.