# Claude Fable 5.2 prompting guide: summary (by Haiku)
Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-2 · fetched 2026-11-13

## TL;DR
- Start at `effort: "high"` (the API default for this model), then re-run your own effort sweep. Most of the capability gain is at `high`; `medium` is the cost-saving step if evals hold. Do not carry over effort settings tuned on Fable 5.1 or Opus 5.5.
- At `low` and `medium` the model tends to end turns early (listing what's left, offering to continue, or stopping at ordinary errors). Add an explicit "keep going through ordinary obstacles" block, but keep your own confirmation step for risky or irreversible actions.
- Behavior is steered by how the prompt frames the work. "You are the owner and long-term maintainer of this codebase" makes it consolidate duplicated code and make unrequested changes; with no ownership statement it tends to add to the existing structure.
- Caution accumulates. The model follows "never" rules as absolute and follows stale memory or notes closely, so it asks for unneeded approvals. Name the exceptions and the approval-free steps explicitly, and delete outdated cautions.
- Legacy prompt lines (delegation pushes, token budgets, "answer directly without deliberating", "treat that answer as done", hard style limits) can now raise cost or cause side effects. Review and remove them before migrating, and write final reports for a reader who has seen none of the tool calls.

## What's distinctive about this model
- **Effort is the main lever.** At `high` and above it thinks before almost every reply, including a greeting. An instruction to answer without deliberating did not stop thinking at `medium` and above. Effort changes thinking, tool calls, latency and tokens, but has little effect on reply length. Length and tone are controlled through the prompt.
- **Easy tasks are handled well at `low`.** On coding evals it scored close to Fable 5.1 at higher effort (per the guide's testing).
- **Early stopping** is more frequent at lower effort on long multistep tasks.
- **Ownership-sensitive.** Without an ownership statement it added to existing structure even when that duplicated code. With the statement it consolidated duplicates in about 75% of runs (vs under 10% without), and output tokens rose about 70 to 80%. It also made unrequested changes (often tests and docs) and sometimes changed existing feature behavior. Its closing PR summary can omit small changes, so the reported diff should be reviewed.
- **Literal rule-following.** It follows "never" rules as absolute. Hard style limits (e.g., "20 words or fewer") can make prose choppy.
- **Cautious when instructed.** With approval rules that listed "sending messages to people outside this conversation," it held back about half of routine requests to colleagues in simulated unattended tasks.
- **Uses subagents more at high effort.** At `xhigh` and `max` it used subagents to review its own work, which Fable 5.1 did not do on the same tasks. Delegation depends on the harness and prompts.
- **Writing quality.** Early testers rated its writing above Fable 5.1 and similar in clarity to Opus 5.5. Final reports are often dense and use semicolons.
- **Preserved thinking.** Editing earlier turns or the system prompt can break the prefix check (400 error).

## Recommended practices
- **Effort:** set `effort` explicitly to `high`; run evals at `medium` and `xhigh`. Use `low` for quick, well-specified, high-volume work and subagents. Reserve `xhigh` for measured gains on hard long-running work, and `max` for measuring a ceiling. Per-message effort (beta) can change level mid-conversation while keeping the cache.
- **Finishing work:** add a "# Finishing work" block. Key lines: "Please avoid stopping while you still owe work the user asked for. Be persistent and do not stop unless needed." and "Diagnose [ordinary obstacles] first... rather than stopping unnecessarily." Deliberate blockers (files marked off-limits, withheld access, safety guardrails) should be left alone and reported. In testing, about half as many sessions ended with unfinished work. Output tokens rose about 30%.
- **Approvals:** say exactly what is approved. Guide example: reply "Yes, run the migration on the staging database" rather than "go" or "continue".
- **Ownership:** for code the agent should restructure, use "You are the owner and long-term maintainer of this codebase, and its design is up to you." Omit it where the user wants minimal change. A scope rule such as "No drive-by refactors" placed earlier in the prompt did not cancel it.
- **Exceptions:** write the exception in place of the absolute. Guide example: "ask for approval before deleting files" rather than "never delete files".
- **Progress visibility:** see the agentic section; supply an opening line, brief updates and a closing recap: "Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own (what you found, what you did, and what's next)..."
- **Subagent tool description:** if the model over-delegates, state in the description what a subagent adds to session cost, when delegation is worth it, and how to brief a subagent. In testing at `high`, a longer description focused on cost roughly halved the number of subagents.
- **Reports and handoffs:** add a "# Communicating clearly" block: "Write for an intelligent but low-context reader." It asks for key points first, plain words, no made-up terms, and short active sentences in the style of ASD-STE100. Adjust length and required details for your readers.
- **Refusals:** if your prompts ask for reasoning in the response text, remove that. To inspect reasoning, use `display: "summarized"` and read the thinking blocks.

## Anti-patterns / things to stop doing
- Sending "continue", "go" or a thumbs-up as approval.
- Writing absolute "never" rules without scope or reason.
- Listing routine actions (e.g., "sending messages to people outside this conversation") among approval-required steps.
- Keeping stale memory or notes such as "never retry" after they stop applying.
- Stopping conditions like "stop and review at the first error" that block autonomous work.
- Standing lines that push delegation, if the model over-delegates.
- Displaying a large token budget to the model (in testing on coding tasks, removing it lowered cost with equivalent performance). This applies to prompt text, not the API task-budget beta.
- "Answer directly without deliberating" or similar think-less lines (no effect at `medium` and above).
- Opus 5.5's "treat that answer as done" lines in chat system prompts. They reduced re-thinking on follow-ups and made the model much less likely to point out its own earlier mistakes.
- Hard prose-limit rules ("one instruction per sentence, in 20 words or fewer").
- Asking the model to write its reasoning in the response text.
- Editing earlier turns or the system prompt mid-conversation (see below).

## Agentic, tool-use and long-context guidance
- **Progress updates.** Between-tool-call updates arrive as `thinking` blocks whose text is empty at the default `thinking.display: "omitted"`. A client that renders only `text` blocks can look silent. Use `display: "updates"` (beta) or `"summarized"`, and render non-empty `thinking` blocks.
- **Silence reminder.** Count consecutive tool-calling steps that produce no user-visible text. When several pass, append a turn-scoped system message (beta): "The user hasn't heard from you in a while: say in a few words what you're doing, then continue." In testing this roughly halved the longest silent stretch. Keep each reminder in later requests; removing or rewording it invalidates thinking blocks after it.
- **Fallback when reminders are unavailable.** "Every few tool calls, give the user a one-line update on where things stand."
- **Preserved thinking and history.** If the harness changes the system prompt, tools or earlier messages, the request can fail (400) or, in Batches, the blocks are dropped. Prefer append-only history. Use context editing or compaction (no earlier turns kept after the summary) to shorten. `prefix_mismatch_behavior: "drop_block"` (beta) avoids the error, but in testing dropping thinking on every turn raised output tokens. Measure before using it.
- **Changes to instructions.** Changes to `system`, `tools` or earlier messages should go into new conversations, or into the newest turn. Mid-conversation system messages are available on some platforms.
- **Subagents.** Check which model subagents use by default; they can be a large share of session cost.
- **Long-running work.** The guide recommends the model for long-running asynchronous tasks where nobody needs frequent updates.
- **Long-context.** Not covered.

## Implications for skill files and agent instruction files
A compliant file:
- [ ] Sets or documents an explicit effort level (`high` default), and says to re-test at `medium`/`xhigh`.
- [ ] Includes a finishing-work block that separates ordinary obstacles from deliberate blockers.
- [ ] Keeps a confirmation step for risky or irreversible actions.
- [ ] States code ownership where restructuring is wanted; omits it where minimal change is wanted.
- [ ] Names approvals specifically; lists only the steps that truly need approval.
- [ ] Phrases limits as scoped exceptions, with reasons, not bare "never".
- [ ] Specifies when subagents are worth using and how to brief them.
- [ ] Asks for an opening line, brief updates and a self-contained recap, if a human is in the loop.
- [ ] Has a low-context-reader block for final reports, with key points first.
- [ ] Does not ask for reasoning in the response text.

A compliant file avoids:
- [ ] Think-less lines, "treat that answer as done" lines, and hard prose-length rules.
- [ ] Standing delegation pushes unless the model under-delegates.
- [ ] Token budgets shown to the model.
- [ ] Stale memory, notes and cautions whose reasons no longer hold.
- [ ] Mid-conversation edits to earlier turns or the system prompt.

## Notable quotes
1. "Start at `high`, the default for Claude Fable 5.2 on the Claude API."
2. "Please avoid stopping while you still owe work the user asked for. Be persistent and do not stop unless needed."
3. "You are the owner and long-term maintainer of this codebase, and its design is up to you."
4. "The model treats an instruction that contains “never” as absolute."
5. "Write for an intelligent but low-context reader."
