OpenClaw is a self-hosted AI assistant framework whose apparent autonomy comes from event-driven architecture, not from the agent continuously "thinking." The agent reacts to inputs; it does not wake up and reason unprompted.

## The Gateway (control plane)

The Gateway is the central hub of the system. It routes incoming events from multiple channels (Slack, Telegram, WhatsApp, Discord, iMessage) to the right agent session, owns all session state, and maintains WebSocket connections with clients. Clients always query the Gateway for state rather than reconstructing it themselves — the Gateway is the single source of truth.

## Session model

Each agent has a primary DM session (typically called "main") and separate sessions for groups, channels, and threads. An optional "secure DM mode" prevents context leakage between users in multi-user scenarios. Session keys define the boundaries of isolation.

## What triggers the agent

OpenClaw's autonomy comes from five input types:

- **Time** — heartbeats on a configurable interval (default 30 minutes)
- **Events** — messages arriving from connected channels
- **Hooks** — internal event-driven automation
- **Webhooks** — triggers from external systems
- **Manual** — direct user input

## The runtime loop

Each agent run follows a fixed cycle: load context → call the LLM → execute tools → persist updates → respond. This loop is intentionally simple and deterministic. [[Context Engineering]] is relevant here — what gets loaded into context at the start of each run directly affects agent quality across long sessions.

## Concurrency and the single-writer rule

The core invariant is that **only one agent run should touch a given session at a time**. This prevents interleaved tool calls, contradictory actions, and corrupted state. Without this guarantee, failures become subtle — half-written files, duplicate replies, inconsistent memory.

This is enforced through a two-stage queue model:

1. **Per-session serialization** via a lane-aware FIFO queue — only one active run per session at a time
2. **Global throttling** via a `maxConcurrent` cap across all sessions

Parallelism is still possible, but only across *different* sessions, never within the same one.

## Queue modes

When a new message arrives while a run is already in flight, the queue mode determines what happens:

- **collect** (default) — coalesce incoming messages into one follow-up turn
- **followup** — wait until the current run ends, then start a new one
- **steer** — inject the new message at the next tool boundary, allowing mid-run course correction
- **interrupt** — abort the active run and start fresh with the newest message

The `steer` mode is the most nuanced: it lets a user redirect an in-progress agent without losing the work already done.

## Transport-level reliability

Inbound deduplication prevents the same message from triggering multiple runs if it arrives more than once from a channel. Rapid text messages are debounced and batched into a single turn, while attachments flush immediately. For side effects (tool calls that write to external systems), idempotency keys ensure that retries don't produce duplicate actions.

## Key insight

Serialization alone is not enough to make an agent system reliable. Side effects must be idempotent, and global concurrency controls must exist alongside per-session serialization. The architecture treats correctness as a first-class constraint — explicit invariants over emergent "agent magic."

## References

- [OpenClaw Architecture Part 1: Control](https://theagentstack.substack.com/p/openclaw-architecture-part-1-control)
- [OpenClaw Architecture Part 2: Concurrency](https://theagentstack.substack.com/p/openclaw-architecture-part-2-concurrency)
