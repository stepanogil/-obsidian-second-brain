Context Engineering is the practice of strategically optimizing what information flows into an agent's context window. The goal is to maintain agent effectiveness across long-running, complex tasks without performance degradation from context bloat.

## Core Principles

Walden Yan of Cognition AI distills context engineering into two principles (see [[Don't Build Multi-Agents - Walden Yan]]):

1. **Share full agent traces, not just individual messages.** Subagents need complete visibility into prior decisions, not just the original task description. Every prior action encodes implicit judgment calls that downstream steps depend on.
2. **Actions carry implicit decisions — conflicting decisions produce bad results.** Parallel agents working from incomplete context will make independent, incompatible choices. Single-threaded execution eliminates this class of failure.

## Techniques

**Context compression** addresses the token limit problem for long-running agents: a specialized LLM summarizes the action history into key decisions and details, producing a compressed trace that fits in the context window while preserving decision continuity.

**Search over injection** is an alternative to loading everything upfront: rather than stuffing context with all prior history, the agent searches for what is relevant at query time. [[OpenClaw]] implements this with a hybrid semantic + keyword memory search, keeping context focused and API costs low.

**Pre-compaction memory flush** is a technique for preventing knowledge loss during compression: before compaction runs, the agent is prompted to write important durable notes to persistent storage. This ensures critical context survives the lossy summarization step.

## Related

- [[Don't Build Multi-Agents - Walden Yan]]
- [[Lessons from Building Claude Code - Seeing like an Agent]]
- [[OpenClaw]]
- [[How Clawdbot Remembers Everything]]

