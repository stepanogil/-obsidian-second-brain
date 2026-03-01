A thread by the Claude Code team on the hardest part of building an agent harness: designing its action space. The core thesis is that building good agent tools is more art than science — it requires paying close attention to model outputs, experimenting, and learning to "see like an agent."

The mental model offered: imagine being given a difficult math problem. What tools would you want? Paper is the minimum, a calculator is better, but a computer (with code execution) is most powerful — if you know how to use it. Design tools shaped to the model's own abilities.

## AskUserQuestion Tool (Elicitation)

The goal was to improve Claude's ability to ask clarifying questions. Three attempts:

1. **Adding questions to ExitPlanTool** — confused Claude by conflating planning and questioning
2. **Custom markdown output format** — Claude was inconsistent; would append extra text or omit options
3. **Dedicated AskUserQuestion tool** — Claude calls it at any point, especially during plan mode; triggers a modal that blocks the agent loop until the user responds. Worked well because Claude seemed to naturally gravitate toward calling it.

## TodoWrite → Task Tool (Evolving with Capabilities)

Early Claude needed a TodoWrite tool to stay on track, plus system reminders every 5 turns. As models improved, reminders became a constraint rather than a help — Claude felt locked into the list. With Opus 4.5 getting better at subagents, a new problem emerged: how do subagents coordinate on a shared todo list?

Solution: replaced TodoWrite with a **Task Tool** that supports dependencies, cross-agent updates, and dynamic modification. Lesson: tools that help weaker models can limit stronger ones. Revisit assumptions constantly.

## Search Interface (Building Context)

Early Claude Code used a RAG vector database to supply context. This was fragile and gave Claude context *passively*. Giving Claude a **Grep tool** let it actively search the codebase and build its own context — a pattern that scales as models get smarter.

This evolved into **progressive disclosure** via Agent Skills: Claude reads skill files, which reference other files it can read recursively. Functionality is added without adding new tools.

## Claude Code Guide Subagent (Progressive Disclosure at Scale)

Claude Code has ~20 tools, and the bar for adding more is high. To handle questions about Claude Code itself (MCPs, slash commands, etc.) without bloating the system prompt, they built a **Claude Code Guide subagent** — prompted when users ask self-referential questions, with specialized instructions for searching docs efficiently. Adding capability to the action space without adding a tool.

## Key Takeaways

- Design tools matched to the model's actual abilities
- Fewer tools is better — each one adds cognitive load
- Use progressive disclosure to expand capability without expanding the tool list
- Constantly revisit which tools are still needed as model capabilities change
- The best tool still fails if the model doesn't naturally want to call it

Related: [[Tool Calling]], [[Context Engineering]]

## References
- [Lessons from Building Claude Code: Seeing like an Agent](https://x.com/trq212/status/2027463795355095314)
