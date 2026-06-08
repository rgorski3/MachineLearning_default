# Beyond the Basics with Claude Code — Key Points

**Video:** [Beyond the basics with Claude Code](https://youtu.be/tuY2ChJIx48)
**Presenter:** Daisy Hollman, Member of Technical Staff at Anthropic
**Event:** Code with Claude 2026

---

## Core Theme

The mechanics that separate basic Claude Code use from real leverage are:
**CLAUDE.md done well**, **MCP tool wiring**, **skills as packaged team knowledge**, and **auto mode used safely**.

---

## CLAUDE.md — Context File Strategy

- CLAUDE.md is a special file placed in the project root that is automatically loaded into context at the start of every session
- Stuffing it with every standard, convention, and acronym feels helpful but you pay token cost for all of it on every turn — leading to worse, more expensive model performance, not better
- Be intentional: stable reference info belongs here; volatile or frequently changing context does not
- KV caching makes early-prompt changes expensive, so keep stable vs. volatile information in the right places

---

## Plugin Abstractions — The Five Primitives

Claude Code has five main plugin abstractions, each with different token overhead tradeoffs:

- **CLAUDE.md** — Markdown context file, auto-loaded every session
- **MCP (Model Context Protocol)** — Wires Claude into external tools via a standard contract; can bloat the system prompt if overloaded
- **Skills** — Reusable, packaged workflows and team knowledge; easiest to create, immediate value
- **Hooks** — Event-driven scripts that run around tool calls, session starts, stop events, and subagent completions; near-zero token overhead since they run externally and inject only when needed
- **Subagents** — Isolated child agents with their own context window, tool list, and permissions; parent delegates, subagent executes in isolation, parent gets only the summary

---

## MCP (Model Context Protocol)

- Makes Claude Code a system-aware agent by exposing external tools through a standard contract
- The agent can call MCP-exposed tools like any other built-in tool
- Overloading MCP can bloat the system prompt — be selective about which tools you expose

---

## Skills

- Package team conventions, workflows, and domain knowledge into reusable slash commands
- The recommended entry point for extending Claude Code — lower barrier than hooks or subagents
- Can be tested and evaluated using Anthropic's Skill Creator and Eval Maker tools

---

## Hooks

- Event-driven scripts that fire on specific Claude Code lifecycle events (tool calls, session start, stop, subagent completion)
- Unlike prompts, hooks execute deterministic code — enforcement is not model-dependent
- Provide near-zero token overhead: logic runs externally, only the result is injected into context
- Best for enforcing rules or running side effects that must always happen regardless of model behavior

---

## Auto Mode and Subagents

- Models like Opus 4.7 can run autonomously for several hours in auto mode
- Daisy Hollman: *"You should be running agents overnight"* — illustrating that supervision of fleets of async agents is now practical
- `/goal` sets a completion condition; Claude keeps running until that condition is verified against the session transcript
- Subagents are the right tool for parallel work or context isolation: code review, test writing, documentation — any repeated task type
- Each subagent gets its own context window, preventing token accumulation from polluting the parent session

---

## Context Management

- Context windows are effectively bounded; long sessions accumulate file reads, tool outputs, and dead ends that crowd out useful space
- Every accumulated token costs money on the next turn
- Strategy: delegate repeated task types to subagents, keep CLAUDE.md lean, use hooks for zero-cost enforcement
- Recommended workflow order: **start with skills → add hooks for deterministic enforcement → use subagents when parallel work or context isolation matters**

---

## Sources

- [Beyond the basics with Claude Code — YouTube](https://www.youtube.com/watch?v=tuY2ChJIx48)
- [Session page — Code with Claude 2026](https://claude.com/code-with-claude/session/sf-beyond-the-basics-with-claude-code)
- [Claude Code Beyond Basics: The Power of Skills & MCP — Why Try AI](https://www.whytryai.com/p/claude-code-ide-skills-mcp)
- [Claude Code CLI: The Complete Guide — Blake Crosley](https://blakecrosley.com/guides/claude-code)
