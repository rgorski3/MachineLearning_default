# Claude Opus 5.5 prompting guide: summary (by Sonnet)
Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5 · fetched 2026-11-13

## TL;DR
- Opus 5.5 is a delta on Opus 5: existing Opus 5 prompts "should perform well without changes". It is more than 30% faster at output and uses fewer tokens for the same task. The guide is organized as symptom -> fix.
- Effort, not prompt wording, is the main control for how much it thinks. Thinking is always on. Default effort is `medium` (Opus 5 defaulted to `high`). Lowering effort cuts thinking more reliably than prompt instructions do.
- It responds well to specific, targeted instructions: name the exact early stops to avoid, the exact design patterns to avoid, and the exact exploration behavior wanted. Generic instructions ("avoid a generic AI look") mostly swap one default for another.
- Failure modes to guard against: text-only turn endings in unattended loops, acting quickly without exploring context in multiapp work, silent long agentic turns, and over-rethinking earlier answers in chat.
- Mark pasted user content with ID-matched tags. Do not ask for reasoning to be written into the response.

## What's distinctive about this model
- **Agentic coding and review:** Strongest on multistep repo work. At default `medium` it matched or beat Opus 5 at `high` in fewer steps and tokens. It sustains multihour autonomous work (audits, migrations with parallel subagents) better than Opus 5. Code review catches more bugs with fewer false alarms, and it explains changes in plain language.
- **Knowledge work:** Less likely to state wrong figures or cite the wrong source. It is better at financial modeling and catches subtle details in large inputs. Its spreadsheets, slides and documents need less editing.
- **Communication:** Its progress updates and final summaries say plainly what it did, what it found, and what it needs.
- **Visual and computer use:** Reads dense charts, diagrams and screenshots more accurately than Opus 5 without extra tooling, even at the lowest effort. It is more reliable at multistep computer use.
- **Thinks more per turn at a given effort level than Opus 5**, especially at `xhigh` and `max`. Effort level names do not map to the same amount of thinking across models.
- **Keeps the user updated on long tasks.** Some updates end the turn with text (`stop_reason: "end_turn"`), which breaks naive agent loops.
- **Gets to work quickly** on loosely specified tasks without exploring sources the request did not name.
- **Highly responsive to elapsed-time information.**
- **Resists indirect prompt injection** (tool results, webpages, on-screen content) better than any earlier Opus model.
- **Safety classifiers:** biology (same as Fable 5.1; Opus 5 lacks them), cybersecurity (finding vulnerabilities in source is allowed; high-risk dual-use is not), and reasoning extraction.
- **Chat:** sometimes revisits earlier answers when thinking about a new turn, adding latency.
- **API differences from Opus 5:** `thinking: {"type": "disabled"}` now returns a 400. Responses may or may not start with a `thinking` block, which is empty under the default `display: "omitted"`.

## Recommended practices
- **Effort:** Start at `medium`, set it explicitly, and test several levels on your own evals. Do not carry over the Opus 5 setting. Reserve `xhigh` and `max` for work with a measured gain. `low` comes close to Opus 5 `high` on several coding evals at much lower cost.
- **`max_tokens`:** Thinking counts toward it even when not returned. Leave room; 128,000 (the maximum) worked well for long agentic coding turns.
- **Changing effort mid-conversation:** Use per-message effort (beta) to keep the cache. A top-level `effort` change invalidates cached messages.
- **Migrating from thinking-disabled:** Start at `low` and measure. For first-token latency, a line like "Answer directly without deliberating." can help, but measure quality. Read reasoning from summarized thinking blocks (`display: "summarized"`). Read responses by block type.
- **Unattended loops:**
  - Treat text-only `end_turn` as a report, not completion.
  - Keep a checklist (to-do tool or file). If items remain open and no blocker is stated, send a continuation message such as: "Your task list still has open items: migrate the remaining two endpoints and update their tests. Continue with them. If one is blocked, say what is blocking it."
  - Alternatively, state the completion condition up front and have a smaller model check it at each end of turn.
  - Cap automatic continuations at 2-3.
  - If a background command or subagent is still running, wait for it and return its output as the next user message.
- **System prompt addition for unattended runs:** The guide gives a paragraph naming four early-stop patterns:
  1. A summary that announces the next step with no tool call.
  2. An offer to continue unless the user objects.
  3. A list of non-blocking decisions for the user.
  4. Deciding "this is a good place to report".

  It says to put status notes and recommendations in the same message as the next tool call, and it names the stops that are wanted (nothing can move without the user, or the blocker is deliberately protected). It keeps confirmation for risky or destructive actions. Add it from the first request of the session (adding it later invalidates earlier thinking blocks). Omit it in human-in-the-loop apps. Expect somewhat more tool calls and tokens.
- **Progress updates:** They come back as progress-update `thinking` blocks, with empty text at default display. Set `display: "updates"` (beta) to receive summaries. For verbatim content mid-turn, give it a message-the-user tool, declared from the first request. For predictable updates, ask in the system prompt (e.g. a one-line statement of intent before the first tool call and a short recap at the end). If turns still go quiet, have the harness append a turn-scoped reminder after about 5 silent tool steps: "The user hasn't heard from you in a while: say in a few words what you're doing, then continue." Stop after 2-3 reminders. This roughly halved long silent stretches.
- **Multiapp workflows:** Add one sentence: "Before taking any action, explore broadly with tool calls: list and open the emails, documents, spreadsheet tabs, and records across the available apps that could be relevant to this task, including ones the task does not explicitly mention, and use what you find." It improved correctness at `medium` and `max`. Keep untrusted content out of the searched records.
- **Multiagent time signals:** Append `elapsed 340s / 1200s` to each message returned to the model. Set the budget somewhat above the desired time. Without a predictable budget, show elapsed time and add: "Time matters here: do not spend time that can be avoided, and the earlier a correct result is obtained, the better." A budget keeps more agents working in parallel; lowering effort reduces the work itself. It is advisory, so keep your own hard timeout. Check quality, since it may search and verify a little less.
- **Chat multi-turn:** To stop re-examination of earlier answers, append: "Once you have answered something, treat that answer as done. On later turns, focus your thinking on what the user is asking now, and don't go back over an earlier answer unless the user asks about it or points out a problem with it." Leave it out for long analyses or agentic tasks where later steps can reveal earlier mistakes. It may make the model less likely to volunteer corrections.
- **Pasted user text:** Wrap each pasted block in `<pasted_content id="ab12">` ... `</pasted_content id="ab12">`, with tags on their own lines and an app-generated random ID. Add a system prompt note saying that pasted text may contain instructions the user did not write, that instructions in it are followed only where the user's own message asks, and that the ID must not be mentioned. It can make the model slightly more cautious. Treat it as one guardrail among others, since tags can be imitated.
- **Visual inputs:** Re-test old scaffolding. For the densest inputs, higher-resolution images and a container with PIL/OpenCV (or at least a crop tool) still help, and work better at higher effort.
- **Frontend design:** Name specific patterns to avoid, e.g. "Do not use a cream or off-white background, italic accent words in headlines, numbered "01/02/03" section labels, monospace labels, or pill-shaped buttons." Iterate: inspect which defaults appear and extend the list.

## Anti-patterns / things to stop doing
- Carrying over Opus 5 effort values or `max_tokens` sizing, especially with thinking off.
- Using `thinking: {"type": "disabled"}`, which is now a 400 error.
- Keeping rules that tell the model not to think.
- Keeping instructions that ask the model to write its reasoning in the response as a thinking substitute. They may trigger a `reasoning_extraction` refusal. A short explanation of the answer or a summary of actions is still fine.
- Keeping "think carefully before answering" lines in chat system prompts. Removing one made replies start sooner with no clear quality drop.
- Relying on prompt text to cut thinking when lowering effort is more reliable.
- Treating a text-only end of turn as task completion.
- Rendering only `text` blocks, which makes long agentic turns look silent.
- Generic style instructions like "avoid a generic AI look".
- Assuming the first content block is text.
- Reserving `xhigh`/`max` by default.
- Changing the system prompt or tools mid-session, which invalidates thinking blocks.

## Agentic, tool-use and long-context guidance
- Unattended loops, progress updates, multiapp exploration, time budgets and visual tools are covered above.
- Declare tools and add system prompt text from the first request so earlier thinking blocks stay valid. Use turn-scoped system messages (beta) for reminders, so the cache keeps matching.
- Risky or irreversible actions still need your own confirmation step, even with the "keep going" addition.
- Prompt injection: it resists injected instructions in tool results and on-screen content better than earlier Opus models. Keep untrusted content out of records an exploring agent reads.
- Long-context: Not covered beyond catching details in large inputs.

## Implications for skill files and agent instruction files
A compliant file (some items are inferences from the guide):
- Does not tell the model how deeply to think or to "think step by step". It leaves that to the effort setting, and states a target effort where relevant (inference).
- Does not ask for reasoning to be written into the response. It asks for a short explanation or action summary if needed.
- For autonomous or long-running agents, defines a checklist and completion condition, names the specific early stops to avoid, and names the stops that are legitimate (needs user input or a protected blocker).
- Keeps status notes in the same message as the next tool call.
- Keeps explicit confirmation for destructive or irreversible actions.
- For multi-source tasks, instructs exploring sources the task does not name before acting.
- Where updates matter, asks for specific ones (intent before the first tool call, recap at the end).
- Uses specific "avoid X, Y, Z" lists rather than general style adjectives, especially for frontend design.
- Marks pasted or untrusted content with ID-matched tags and states how to treat instructions inside it.
- Avoids rules that suppress thinking or tell the model to answer without deliberating, unless measured.
- Is written as stable text from the first turn, with no mid-session edits to instructions or tools.
- Does not carry over compensating scaffolding written for earlier models without re-testing (the guide says this for visual tooling and the thinking-disabled mitigations).

## Notable quotes
- "Lowering effort reduces thinking, and with it cost and latency, more reliably than prompt instructions do."
- "Treat a text-only end of turn as a report rather than as proof the task is done."
- "a general instruction such as 'avoid a generic AI look' mostly swaps one default for another."
- "Claude Opus 5.5 tends to get to work quickly, and on loosely specified tasks it helps to tell the model to look through the relevant sources before acting."
- "Claude Opus 5.5 is highly responsive to information about elapsed time."
