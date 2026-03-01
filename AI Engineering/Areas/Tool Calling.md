Tool Calling is the mechanism by which AI agents interact with external systems and execute actions. 

In the Claude API, tools are the primitives that define an agent's **action space** — the full set of things it can do.

Tool calling can be implemented using several primitives in the Claude API: bash execution, skills, and code execution. The design of an agent's tool set is a critical engineering decision that shapes how effectively the model can operate.

Key design principles for tool calling:
- **Match tools to model abilities** — give the model tools shaped to what it can actually use well
- **Keep the action space small** — each additional tool adds cognitive overhead for the model
- **Revisit assumptions as capabilities evolve** — tools that once helped a weaker model may constrain a stronger one
- **Use progressive disclosure** — instead of adding more tools, expose context incrementally through search and skill files

See [[Lessons from Building Claude Code - Seeing like an Agent]] for concrete examples of how tool design evolved in practice (AskUserQuestion, TodoWrite → Task Tool, Grep for codebase search).

## References
- [Lessons from Building Claude Code: Seeing like an Agent](https://x.com/trq212/status/2027463795355095314)
