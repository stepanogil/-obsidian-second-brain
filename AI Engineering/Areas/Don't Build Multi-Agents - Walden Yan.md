Walden Yan of Cognition AI argues that multi-agent architectures are currently unreliable and that building effective agents comes down to two core principles of [[Context Engineering]].

## The Core Argument

Multi-agent systems fail not because the individual agents are bad, but because parallel agents make independent decisions that conflict with each other. The architecture itself disperses decision-making in a way that produces inconsistent outputs. The solution is not better agents — it is better context discipline.

## The Two Principles of Context Engineering

**Principle 1: Share context — share full agent traces, not just individual messages.**

Subagents need complete visibility into all prior decisions and reasoning, not just a summary of the original task. When you pass only the task description to a subagent, it loses all the implicit decisions baked into the steps already taken. Sharing full traces ensures downstream agents work from the same interpretive foundation.

**Principle 2: Actions carry implicit decisions, and conflicting decisions produce bad results.**

Every action an agent takes encodes a judgment call. When two parallel agents work independently on the same task, each makes its own judgment calls — and those judgments will often be subtly incompatible. The result is components that don't fit together.

## The Flappy Bird Illustration

The article walks through a concrete failure mode: instruct two subagents to build a Flappy Bird game in parallel — one handles art, one handles gameplay. Without shared context, one agent might build a Mario-style background while the other creates a bird designed for a different aesthetic. Even if both agents read the original spec, each makes independent stylistic decisions that clash.

The fix isn't coordination messages between agents. It's keeping all decisions on a single thread so every subsequent action is informed by everything that came before.

## Recommended Architectures

**Single-threaded linear agents** are the default recommendation. One agent, one continuous context, sequential execution. Context never diverges. This is the simplest and most reliable approach for most tasks.

**Context compression** is the extension for long-running tasks where the context window would overflow. A specialized LLM summarizes the action history into key decisions and relevant details, and that compressed summary becomes the working context going forward. This preserves decision continuity while managing token limits.

## Real-World Examples

The article points to [[Lessons from Building Claude Code - Seeing like an Agent|Claude Code]] and edit-apply models as systems that follow these principles in practice — they avoid parallel subagent delegation and keep decision-making centralized on a single thread.

## Current State and Future

Multi-agent collaboration produces fragile systems today because dispersed decision-making makes it nearly impossible to maintain consistent judgment across agents. The author expects this to improve as agents develop better communication primitives — but for now, single-threaded agents with disciplined context sharing are the more reliable architecture.

## References

- [Don't Build Multi-Agents — Cognition AI](https://cognition.ai/blog/dont-build-multi-agents)
