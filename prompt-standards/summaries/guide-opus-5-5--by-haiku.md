# Claude Opus 5.5 prompting guide: summary (by Haiku)
Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5 · fetched 2026-11-13

## TL;DR
- Opus 5.5 is a drop-in successor to Opus 5 (existing prompts should work), but it defaults to `medium` effort, not `high`, and thinking is always on. Set effort explicitly and measure.
- Thinking cannot be disabled: `thinking: {"type": "disabled"}` returns a 400 error. Prompts that depended on no-thinking mode need changes.
- In unattended agents, a text-only `end_turn` is a progress report, not proof of completion. Keep a checklist and add continuation logic (capped at two or three).
- Use `max_tokens` large enough for thinking plus the reply (128,000 used in testing), and reserve `xhigh`/`max` for measured gains.
- Remove "think carefully" and "show your reasoning in the response" instructions. Effort and summarized thinking are the controls.

## What's distinctive about this model
- **Speed and token use:** Output is more than 30% faster than Opus 5, and it tends to finish the same task with fewer tokens.
- **Thinking:** Always on. At a given effort level it thinks more per turn than Opus 5, especially at `xhigh` and `max`. Lowering effort reduces thinking more reliably than prompt instructions do.
- **Strengths:** Multistep agentic coding in real repos (at `medium` it matched or beat Opus 5 at `high`), code review (more bugs caught, fewer false alarms), long autonomous runs, knowledge work (fewer incorrect figures, better financial modeling, catches details in large inputs), and visual inputs (charts, diagrams, screenshots, computer use). Computer use at default effort matched Opus 5's success rate at a much higher effort.
- **Prompt-injection resistance:** Better than any earlier Opus model against instructions arriving via tool results, webpages, and on-screen content.
- **Safeguard classifiers:** Biology, cybersecurity, and reasoning extraction. Biology safeguards match Claude Fable 5.1's and are absent from Opus 5. Vulnerability finding in source code is allowed; high-risk dual-use cyber work is not.
- **Progress updates:** Between tool calls it writes short progress notes, returned as `thinking` blocks whose text is empty at the default `thinking.display`.
- **Failure modes the guide names:**
  - Ending a turn early with a summary that announces the next step, or an offer to continue, instead of taking the step.
  - In multi-turn chat, re-examining earlier answers while thinking about a new message, adding latency.
  - Generic frontend styling when no design direction is given.
  - Under time pressure, slightly less searching and verification (inference from the time-budget section).

Differences from Opus 5 the guide states: default effort `medium` (Opus 5 is `high`); thinking-disabled returns a 400; biology classifier added; stronger visual reading even at low effort.

## Recommended practices
- **Set effort explicitly.** Start at `medium`, test `low` and others against your own evals. Use `low` for cost-sensitive coding; several coding evals show it close to `medium`.
- **Size `max_tokens` for thinking.** Thinking counts toward it even when not returned. A limit sized for Opus 5 with thinking off can truncate replies.
- **Change effort per message, not via the top-level field, mid-conversation** (per-message effort is beta). Changing the top-level value invalidates the cache.
- **If migrating a thinking-disabled integration:** start at `low`, measure, and move to `medium` if quality drops. Optionally add "Answer directly without deliberating." Read responses by block type, since the first block may be an empty `thinking` block.
- **Read reasoning from summarized thinking**: `display: "summarized"`. You may still ask for a short explanation of the answer or a summary of actions taken.
- **Keep a checklist for multipart tasks** the model updates (a to-do tool or file). When a turn ends with items open and no stated blocker, send a continuation message such as: "Your task list still has open items: ... Continue with them. If one is blocked, say what is blocking it."
- **Wait for background work.** If a background command or subagent is still running, return its output as the next user message before treating the task as done.
- **Name the early stops you don't want** and the stops you do want (e.g., nothing can advance without the user's input). The guide supplies a full unattended-run paragraph: end turns only when nothing can move without the user; put status notes in the same message as the next tool call.
- **Mark pasted text.** Wrap pasted blocks in matching `<pasted_content id="...">` tags with an app-generated random ID, and add a system note that instructions inside those tags are not the user's unless the user's own message asks for them. Treat as one guardrail among others.
- **Explore broadly in multiapp workflows.** "Before taking any action, explore broadly with tool calls: list and open the emails, documents, spreadsheet tabs, and records across the available apps that could be relevant to this task..."
- **Tune progress updates.** Set `display: "updates"` (beta), state the desired update frequency in the system prompt, and optionally have the harness inject a reminder after about five silent tool-calling steps.
- **Chat system prompts:** remove "think carefully" lines. To stop re-litigating earlier answers, add: "Once you have answered something, treat that answer as done..." (omit this in long analyses or multistep agentic work where later steps may reveal earlier mistakes).
- **Visual inputs:** re-test any scaffolding built for earlier models. For dense inputs, use higher resolution and, for agentic setups, a container with PIL/OpenCV or a cropping tool. Raising effort helps with technical drawings more than charts.
- **Frontend:** name specific patterns to avoid rather than "avoid a generic AI look", then check the first output and extend the list. The guide's example bans cream or off-white backgrounds, italic accent words, "01/02/03" labels, monospace labels, and pill-shaped buttons.
- **Time budgets (multiagent):** have the harness append `elapsed Ns / Ms` to each message, set the budget somewhat above the time you want spent, and keep your own hard timeout (the budget is advisory).

## Anti-patterns / things to stop doing
- Carrying over an Opus 5 `effort` value (default was `high`) and expecting the same behavior. Expect longer turns and more output.
- Sizing `max_tokens` as if thinking were off.
- Setting `thinking: {"type": "disabled"}`. It returns a 400.
- Asking the model to write its reasoning into the response text. It may be declined with `reasoning_extraction`, which server-side fallback does not retry.
- Treating a text-only end of turn as task completion in unattended loops.
- Ending chat or agent turns with offers to continue, decision lists the user doesn't need to answer, or "good place to report" stops when work the user asked for is still owed (guide's sample instruction lists these four).
- Writing "think carefully before answering" in chat system prompts (guide: removing it made replies start sooner with no clear quality loss).
- Using a general style instruction for frontend work; it swaps one default for another.
- Adding tools, or changing the system prompt, partway through a session. This invalidates earlier thinking blocks. Declare tools from the first request; add mid-conversation reminders as appended, turn-scoped messages instead.
- Relying on prompt instructions to cut thinking when lowering `effort` is the more reliable lever.
- Treating a time budget as a hard stop.

## Agentic, tool-use and long-context guidance
- **Unattended loops:** Track completion against a checklist or a separate checker model at each end of turn; cap automatic continuations at two or three so stuck runs end and can be reviewed. Keep human confirmation for risky or irreversible actions. Leave the continuation paragraph out of human-in-the-loop apps.
- **Harness messages:** Expect progress notes between tool calls; render `display: "updates"` if the client shows only `text` blocks, or the run looks silent.
- **Tool declaration:** Declare any tool the model may need to hand the user verbatim content (such as a code snippet) from the first request; reserve it for that purpose.
- **Multiapp:** The model acts on what it finds, so keep untrusted content out of the records it searches.
- **Subagents and parallelism:** Time signals speed up multiagent teams. A tighter budget keeps more agents parallel; effort reduces work itself.
- **Long context:** Not covered. The guide gives no long-context-specific guidance.

## Implications for skill files and agent instruction files
A compliant file:
- [ ] Specifies effort explicitly and says `medium` is the baseline to test against; reserves `xhigh`/`max` for measured needs.
- [ ] Assumes thinking is always on; contains no `thinking: disabled` settings or "no thinking" rules.
- [ ] Sets `max_tokens` with thinking headroom (128,000 cited for long agentic turns).
- [ ] Does not ask for reasoning to be written into the response; uses summarized thinking if reasoning must be read.
- [ ] Has a completion condition and a checklist for multipart work; defines continuation behavior with a cap.
- [ ] Says what to do with background tasks before declaring completion.
- [ ] Names the early stops to avoid (summary-then-stop, offers to continue, decision lists with no blocker).
- [ ] Omits "think carefully" in chat system prompts; optionally includes the "treat earlier answers as done" line (chat only).
- [ ] Wraps pasted content in ID-tagged blocks where user content is mixed with external text.
- [ ] Uses specific patterns to avoid for frontend work, not a general "avoid AI look".
- [ ] Keeps tool definitions and system prompt stable within a session.

A non-compliant file tends to:
- Carry Opus 5 settings forward unchanged (high effort defaults, thinking-off workarounds).
- Treat a text-only reply as the end of an unattended run.
- Ask for visible chain-of-thought.
- Add tools or rewrite the system prompt mid-session.
- Rely on "avoid X" without listing X.

## Notable quotes
1. "Start at `medium`, the default on Claude Opus 5.5 (Claude Opus 5 defaults to `high`)"
2. "Lowering effort reduces thinking, and with it cost and latency, more reliably than prompt instructions do."
3. "Treat a text-only end of turn as a report rather than as proof the task is done."
4. "Once you have answered something, treat that answer as done."
5. "Before taking any action, explore broadly with tool calls"
