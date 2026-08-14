# Agent Note: subagent_with_model model/provider override

Status: implemented

## Problem

The native `subagent` tool cannot pick a child model/provider per call; callers need to choose a model explicitly for one delegation while still inheriting defaults when they don't care.

## Decision

`lib/index.js` registers the `subagent_with_model` tool (schemastery `Config`: `subagentProvider` default `spawn`, `toolName` default `subagent_with_model`, `backgroundMode` `"one-shot" | "continuable"` default `"one-shot"`, `maxDepth` default `3`). Parameters are `model` (required), `provider` (optional), `description` (required), `prompt` (required), and `run_in_background` (optional boolean).

The override semantics are: **only explicitly-passed values are forwarded.** `agentOptions` spreads `provider` and `model` only when each argument is not `undefined` (`...(args.provider !== undefined ? { provider: args.provider } : {})` and the same for `model`), so an omitted `provider` inherits the parent's provider and an omitted `model` inherits the parent's model. `run_in_background` defaults to the configured `backgroundMode` (`continuable` → background by default; `one-shot` → foreground by default).

Execution takes one of three paths, all through the native `ctx.subagents` service:

- **Continuable background** (`runInBackground && continuable`) → `ctx.subagents.startContinuable(...)` → returns `{ kind: "continuable", subagentId: childId }`.
- **One-shot background** (`runInBackground && !continuable`) → `ctx.get("jobs").start(...)` wraps `ctx.subagents.start(...)` with an `AbortController` and `settleStart` → returns `{ kind: "background", jobId }`.
- **Foreground** → `ctx.subagents.start(...)`, awaits `run.result`, throws on any non-`completed` `stopReason` (including the partial output), and returns `{ kind: "foreground", runId, output }` before `run.dispose()`.

The tool registers lazily: it mounts when the named provider appears (`subagent/provider-added`) and disposes on `subagent/provider-removed`, and `continuable` mode fails fast if the provider lacks `prepareContinuable`.

## Alternatives considered

- **Reuse the native `subagent` tool with a config-level `agentOptions`** — rejected: a config value applies to every call and cannot be set per delegation, which is the whole point.
- **Always send `provider` (defaulting to the parent's provider)** — rejected: that pins the provider at call time and cannot express "inherit whatever the parent is using now".
- **Make `provider` required** — rejected: it breaks the inherit-by-default ergonomics documented in the README and forces callers to know the parent's provider.

## Consequences

- Callers get an explicit override only when they pass one, and inheritance otherwise; `model` stays required so delegations are deliberate.
- `backgroundMode` changes both the default `run_in_background` and the tool's description text, so the two must be kept consistent.
- The one-shot background path depends on `@deepseek-ai/dsh-jobs`; without it the tool errors at call time rather than silently degrading.
