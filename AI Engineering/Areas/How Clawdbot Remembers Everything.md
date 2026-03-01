This article analyzes the persistent memory architecture of [[OpenClaw]] (also called Clawdbot), an open-source local AI assistant with over 32,600 GitHub stars. The focus is on how the system maintains 24/7 context retention — remembering conversations and building on previous interactions indefinitely — while keeping all data local and user-controlled.

## What the model sees on each request

Every agent run loads four layers of context in order:

1. **System prompt** — static and conditional instructions defining agent capabilities and available tools
2. **Project context** — user-editable bootstrap Markdown files (AGENTS.md, SOUL.md, USER.md) injected into every request
3. **Conversation history** — messages, tool calls, and compaction summaries
4. **Current message** — the incoming input

## Context vs memory: the core distinction

This distinction drives all architectural decisions in the system.

**Context** is everything the model sees for a single request. It is ephemeral (exists only for this request), bounded (limited by the model's context window, e.g. 200K tokens for Claude), and expensive (every token incurs API cost and affects latency).

**Memory** is what is stored on disk. It is persistent (survives restarts), unbounded (can grow indefinitely), cheap (no API cost to store), and searchable (indexed for semantic retrieval).

The system's goal is to keep context focused and relevant while memory handles unbounded long-term storage. This is a practical application of [[Context Engineering]]: rather than injecting everything into context, the agent searches for what is relevant.

## Memory tools

Two specialized tools give the agent access to stored memory:

**memory_search** — semantically searches MEMORY.md and all files in `memory/`. The agent's AGENTS.md instructs this as a mandatory recall step before answering questions about prior work, decisions, dates, people, preferences, or todos. Parameters include `query`, `maxResults`, and `minScore` (default threshold: 0.35).

**memory_get** — reads specific lines from a memory file by path and line range. Used after `memory_search` identifies the relevant file and location.

There is no dedicated `memory_write` tool. The agent writes to memory using the same standard write and edit tools used for any file. Because memory is plain Markdown, users can also edit it manually — the index rebuilds automatically on changes.

## Two-layer memory storage

Memory lives in the agent's workspace (default: `~/clawd/`):

```
~/clawd/
├── MEMORY.md              ← Layer 2: Long-term curated knowledge
└── memory/
    ├── 2026-01-26.md      ← Layer 1: Today's daily log
    ├── 2026-01-25.md
    └── ...
```

**Layer 1 — Daily logs** (`memory/YYYY-MM-DD.md`): Append-only notes the agent writes throughout the day. Written whenever the agent wants to remember something, or when explicitly told to. Relatively raw, timestamped entries for that day's activity.

**Layer 2 — Long-term memory** (`MEMORY.md`): Curated, persistent knowledge. The agent writes here for significant events, decisions, opinions, and lessons learned. This is the primary target for high-level preferences, key contacts, and important decisions.

## Hybrid search

Memory is indexed in a SQLite database using two extensions:
- **sqlite-vec** — enables vector similarity search directly in SQLite, no external vector database required
- **FTS5** — SQLite's built-in full-text search engine, powering BM25 keyword matching

Two search strategies run in parallel on every `memory_search` call:
- **Vector search (semantic)** — finds content that means the same thing as the query
- **BM25 search (keyword)** — finds content containing exact tokens from the query

Results are merged with weighted scoring: `finalScore = (0.7 × vectorScore) + (0.3 × textScore)`. The 70/30 split prioritizes semantic recall while BM25 catches exact terms that vectors may miss — names, IDs, dates, identifiers. Results below the `minScore` threshold are filtered out. All weights are configurable.

## Multi-agent memory isolation

Multiple agents each get a separate workspace directory and a separate SQLite index. The Markdown source files live in each agent's workspace (source of truth); the SQLite indexes (derived data) live in a shared state directory keyed by `agentId + workspaceDir`. No cross-agent memory search occurs by default.

This isolation enables separate contexts: a "personal" agent for WhatsApp and a "work" agent for Slack, each with distinct memories and personalities. An agent could theoretically access another workspace via absolute paths, but strict sandboxing mode prevents this.

## Compaction

Every model has a context window limit. When a long conversation approaches that limit, compaction summarizes older turns into a compact entry while preserving recent messages intact. The compacted summary is written back to the session's JSONL transcript, so future sessions start with the compacted history rather than the full raw conversation.

Compaction can be:
- **Automatic** — triggers when the conversation approaches the context limit
- **Manual** — triggered with `/compact`, optionally with a focus instruction (e.g., "focus on decisions and open questions")

Unlike some in-memory optimizations, compaction persists to disk.

## Pre-compaction memory flush

LLM-based compaction is lossy — important information can be summarized away and lost. The memory flush counters this: when context approaches a configurable token threshold (`softThresholdTokens`), the agent is prompted to write important, durable notes to the current day's memory file before compaction runs. This ensures critical context survives even an aggressive compaction.

The flush is configurable in `clawdbot.yaml` or `clawdbot.json`, including the threshold, system prompt, and the specific instruction given to the agent for what to write.

## Tool result pruning

Tool results (e.g., exec command output) can be extremely large — a single command might return 50,000 characters of logs. Pruning trims old tool result content in-memory to reduce context size. Unlike compaction, pruning does not modify the on-disk JSONL transcript; the full outputs remain on disk. It is irreversible within the session.

**Cache-TTL pruning** addresses a specific cost problem with Anthropic's prompt prefix caching. Anthropic caches prompt prefixes for up to 5 minutes, reducing cached token costs by ~90%. When a session goes idle past the TTL, the cache expires and the full conversation must be re-cached at full "cache write" pricing. Cache-TTL pruning detects expiry and trims old tool results before the next request, reducing the prompt size that needs to be re-cached and lowering costs.

## Session lifecycle and memory hooks

Sessions reset on configurable schedules (default: daily). When starting a fresh session via `/new`, a session memory hook can automatically save the previous session's context to the memory files before resetting. This ensures prior context remains searchable via `memory_search` in future sessions — the session boundary does not create a knowledge gap.

## Key design principles

The article concludes with four principles underlying the memory architecture:

1. **Transparency over black boxes** — memory is plain Markdown: readable, editable, version-controllable, no proprietary formats
2. **Search over injection** — retrieve only what is relevant rather than stuffing context with everything; keeps context focused and costs down
3. **Persistence over session** — important information is written to disk before compaction can destroy it; files outlive any conversation
4. **Hybrid over pure** — vector search alone misses exact matches; keyword search alone misses semantics; hybrid gives both

## References

- [How Clawdbot Remembers Everything](https://x.com/manthanguptaa/status/2015780646770323543)
