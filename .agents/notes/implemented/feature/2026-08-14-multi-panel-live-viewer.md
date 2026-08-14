# Agent Note: Multi-panel live viewer

Status: implemented

## Problem

Users need to watch several subagents at once — live token-by-token output, plus per-subagent metadata — without a fixed single pane or a full session view.

## Decision

`lib/client.js` builds a client-side store (`createStore`) holding one panel per subagent session: it buffers live events per session (capped at `MAX_EVENTS = 5000`), backfills history via `api.sessions.history` / `api.subagents.history`, and merges buffered events over the backfill by sequence.

The UI is rendered through two slot injections:

- **`shell.overlay`** renders multiple floating panels (`PanelWindow`) plus a notification stack. Panels are independently draggable (pointer-captured drag on the header), resizable from the bottom-right and bottom-left corners, z-ordered (`bringToFront`), and positioned with a clamp + overlap-aware cascade so new panels don't stack on existing ones.
- **`conversation.view`** renders the **Subagents** tab: a card grid grouped into active/inactive and sorted by `lastActivity`, each card showing model, tokens, steps, context %, duration, and relative time. Cards support **drag-to-pop-out**: a pointer-captured drag with a ghost preview opens the panel exactly where it's dropped.

Text is internationalized through `ctx.locale.register("subagent-max", "zh"/"en", ...)` and `T(key)`. `lastActivity` is tracked per session in memory, persisted (debounced) to `localStorage`, and backfilled from history via `seedLastActivity` when missing.

## Alternatives considered

- **A single fixed viewer panel** — rejected: it cannot watch parallel subagents at once, which is the core need.
- **Reuse DSH's built-in session view only** — rejected: it doesn't give per-subagent live token streaming plus the card metadata (tokens/steps/context%/tps) this tab shows.
- **A server-side activity index** — rejected: client-side tracking plus history backfill is simpler and sufficient for a viewer; a server index would be over-engineering.

## Consequences

- Multiple live panels require explicit bookkeeping — z-order, clamped positions, drag/resize pointer listeners — and cleanup on unmount: interval/timer removal, `AbortController.abort()`, listener removal, and `store.dispose()`.
- Model display depends on each session's `request/header`, so some cards show no model.
- `lastActivity` is a client-side cache (with `localStorage` persistence), so a fresh load can briefly fall back to the session's `updatedAt` until history is seeded.
