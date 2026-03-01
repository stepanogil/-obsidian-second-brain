OpenClaw is an open-source, self-hosted personal AI assistant framework (MIT licensed) created by Peter Steinberger, also referred to as Clawdbot. Unlike cloud-hosted assistants, it runs locally on the user's machine and integrates with existing chat platforms (Discord, WhatsApp, Telegram, Slack, iMessage). All context, configuration, and memory remain under user control.

## What it does

OpenClaw handles autonomous real-world tasks: managing emails, scheduling calendar events, handling flight check-ins, and running background jobs on a schedule. Its architecture is event-driven rather than always-on — the agent reacts to inputs rather than continuously reasoning. Input sources include time (configurable heartbeats), channel events, internal hooks, external webhooks, and direct user messages.

## Key properties

- **Local-first**: agent configuration, memory, and context all live in plain Markdown files on disk — readable, editable, and version-controllable
- **Multi-channel**: a single agent is accessible across multiple platforms via a central Gateway that routes events and manages sessions
- **Persistent memory**: a two-layer system combining append-only daily logs (`memory/YYYY-MM-DD.md`) and a curated long-term knowledge file (`MEMORY.md`)
- **Hybrid search**: memory is indexed with SQLite (sqlite-vec + FTS5) enabling both semantic vector search and BM25 keyword matching without an external vector database
- **Multi-agent isolation**: each agent instance has its own workspace and SQLite index, enabling separate contexts (e.g., a personal agent for WhatsApp and a work agent for Slack)

## Architecture

Two major subsystems work together:

- **Control plane** — the Gateway routes events to the correct agent session, enforces single-writer serialization via a lane-aware FIFO queue, and manages concurrency with a global `maxConcurrent` cap. See [[OpenClaw architecture]].
- **Memory system** — plain Markdown files indexed for hybrid search, with compaction (LLM-based summarization of old turns), a pre-compaction memory flush (persisting important context before compaction runs), and tool result pruning. See [[How Clawdbot Remembers Everything]].

Each agent run follows a fixed cycle: load context → call the LLM → execute tools → persist updates → respond. What gets loaded into context at the start of each run is the core [[Context Engineering]] challenge for long-running personal assistants.

## Why it matters

OpenClaw demonstrates that persistent, capable AI assistance does not require cloud infrastructure. Transparency is a first-class design principle: because memory is plain Markdown, users can inspect exactly what the agent knows, correct mistakes directly, and version-control their entire agent context.

## Related

- [[OpenClaw architecture]]
- [[How Clawdbot Remembers Everything]]
- [[Context Engineering]]
