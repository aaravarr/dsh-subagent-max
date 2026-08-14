# Agent Note: Dual-face host/client contract

Status: implemented

## Problem

The package ships two faces that run in different runtimes and cannot share code directly: a host face (`lib/index.js`) that registers a model-facing tool inside the Cordis host, and a client face (`lib/client.js`) that renders a Web UI in the browser. They must agree on a stable boundary — what each owns, and how the client observes subagent activity without reaching into host internals.

## Decision

The host face owns delegation: `lib/index.js` is a Cordis plugin (`name: "dsh-subagent-max"`, `inject: ["tools", "subagents"]`) that registers the `subagent_with_model` tool via `ctx.tools.register(defineTool(...))` and forwards every call into the native `ctx.subagents` service. It knows nothing about the UI.

The client face owns presentation: `lib/client.js` loads through `window.__ModuleLoader__.load({ id: "@aaravarr/dsh-subagent-max", factory })` (its `id` equals `package.json` `name`) and declares `inject: ["sessions", "connection", "slots", "locale"]`. It reads live events over DSH's public event streams — `api.events.mux({}, signal)` for `session/event` frames and `api.events.host({}, signal)` for host frames (`host/session-added`, `host/agent-error`, `host/session-status`) — and renders through two slot injections: `ctx.slots.inject("shell.overlay", ...)` for the floating panels and notifications, and `ctx.slots.inject("conversation.view", ...)` for the **Subagents** tab.

The contract between the faces is deliberately narrow: they share only the package name/id and the subagent session model. The client never assumes the tool name or the configured `subagentProvider`; it discovers model/provider by reading each session's `request/header` from history, and it filters `session/event` and host frames to subagent sessions by origin/parentId rather than coupling to host config.

## Alternatives considered

- **A private host→client push channel** — rejected: DSH already exposes `api.events.mux`/`api.events.host`, so a bespoke protocol would duplicate the event bus and add a surface to maintain.
- **Polling `ctx.sessions.list` snapshots only** — rejected: polling cannot stream token-by-token output and would re-implement the live stream the events API already provides.
- **Hardcoding the tool name (`subagent_with_model`) in the client** — rejected: `Config.toolName` is configurable, so the client must not assume a fixed name.

## Consequences

- Host and client evolve independently and only renegotiate when the package name/id or the session shape changes.
- The client must tolerate noise: it filters incoming events and host frames to subagent sessions and keeps working when a session's `request/header` (and therefore model display) is absent.
- The client's correctness depends on DSH's public event and slot APIs staying additive, which the hard constraints in `AGENTS.md` already require.
