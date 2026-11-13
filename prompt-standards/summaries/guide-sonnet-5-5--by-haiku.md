# Claude Sonnet 5.5 prompting guide: summary (by Haiku)
Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5 · fetched 2026-11-13

## TL;DR
- Effort is the main control, and its levels are recalibrated: the same level does not mean the same amount of thinking as on Claude Sonnet 5. Start at `high` (the API default) unless the work is agentic or latency-sensitive, then use `medium`. Re-sweep against your own evals rather than carrying over Sonnet 5 settings.
- Scope and initiative vary with effort. At `low`/`medium` it may stop to check in on long coding tasks. At `xhigh`/`max` it adds unrequested tests, docs, and extra review rounds. Steer both with explicit system-prompt paragraphs the guide supplies.
- At `low` effort it can report a change as done without running a real check. Add an explicit verification paragraph.
- Reduce thinking by lowering effort, not by asking in the prompt. JSON reasoning tasks need `Think the problem through before you answer.` (adaptive thinking) or structured outputs.
- Keep user text out of `tool_result` blocks. The model may treat such text as a prompt injection.

## What's distinctive about this model
- Existing Sonnet 5 prompts should perform well unchanged. For the hardest long-horizon work the guide recommends an Opus model.
- At `low` effort it keeps thinking short and can skip verifying a change. At `low` and `medium`, on long agentic tasks, it is more likely to stop and check in before finishing.
- It tends to add tests, documentation, and small supporting files that fit repository conventions, at every effort level and more at higher effort.
- At `xhigh` and `max` it is especially thorough: it starts its own review and verification rounds, may launch subagents if the harness provides them, and may make related fixes it noticed.
- It is trained to resist indirect prompt injection. It sometimes treats a genuine mid-task user message as injected text.
- It sometimes answers from training knowledge when a search would catch details that have changed (what is allowed, required, or charged).
- It occasionally calls a declared tool with different letter case (`bash` for `Bash`) or a slightly different parameter name.
- With structured outputs at `low`/`medium`, it occasionally keeps thinking until `max_tokens`.
- It runs safety classifiers that can return `stop_reason: "refusal"` with categories `cyber`, `bio`, `frontier_llm`, `reasoning_extraction`, and `general_harms`.
- Differences from earlier or sibling models beyond recalibrated effort and the behaviors above: Not covered.

## Recommended practices
- Effort: `medium` for well-specified agentic coding, `high` for harder or longer work; `medium` or `low` for chat and latency-sensitive work. Use `xhigh`/`max` only where you have measured a quality gain.
- Set `max_tokens` with room for thinking and the reply (thinking counts toward it). For agentic coding, use 128,000 and stream.
- Use per-message effort changes (beta) to vary effort mid-conversation without invalidating the cache. Changing top-level `effort` invalidates the cache.
- To keep work going without changing effort, add: "Keep working until everything the user asked for is done, and only stop to ask when you can't go on without the user or before a risky step."
- To limit scope, add: "When the work the user asked for is done and checked, stop and report. Don't add features, tests, files, docs, or refactors that weren't asked for."
- At `xhigh`/`max`, to stop self-started review: "Don't start extra rounds of review or hardening on your own, and don't launch reviewer subagents unless the user asked for a review."
- For open-ended requests where you want ideas first: "When the user asks for ideas, options, or a plan, give them that and stop."
- For changed facts, remove language like "only use tools when strictly necessary" or "minimize tool calls", then add: "Use the search tool to check specifics that may have changed since your training... even when you feel confident."
- For code changes, add a verification paragraph requiring a real check (tests, type-checker, build, or the changed command), not a syntax-only check, and stating which check was not run if none can run.
- For progress, remove "hold all findings for the final response", then state update points in the system prompt (for example, a line before the first tool call and a recap at the end).
- For JSON reasoning tasks: prefer structured outputs. Otherwise add "Think the problem through before you answer." with adaptive thinking, or use `xhigh` with adaptive thinking.
- For dense charts and technical drawings, give the model crop, zoom, or code-execution tools on the image.
- If you need an explanation, ask for a short explanation of the answer or a summary of actions taken, not the internal reasoning.

## Anti-patterns / things to stop doing
- Asking the model in the system prompt to think less. The guide says this does not reliably reduce thinking.
- Discouraging tool use ("only use tools when strictly necessary", "minimize tool calls").
- Leaving "hold all findings for the final response" in place when you want progress updates.
- Putting user text inside a `tool_result` block, or putting a harness notice and the user's words in the same block.
- Frequent harness text or token countdowns after every tool result in interactive sessions. Reminders should be occasional; the guide suggests stopping after the second or third if the turn stays quiet.
- Asking the model to include its reasoning in the response text (invites `reasoning_extraction` declines).
- Using `between_tools` with `xhigh`/`max` (400 error), with `display`, `budget_tokens`, or `block_binding` (400 error), or with a per-message effort that differs from the effective level (400 error).
- Adding "don't think" instructions under `between_tools`; the guide says they make internal XML tags in visible output more likely.
- Sizing `max_tokens` for a request without thinking.
- Treating a response with `stop_reason: "max_tokens"` as success because its JSON parses.
- Assuming the first content block is text. Responses can start with an empty `thinking` block.
- Sending back only summaries of `thinking` blocks; send them back unchanged.
- Running routine work at `xhigh`/`max`.

## Agentic, tool-use and long-context guidance
- Progress updates: notes longer than a sentence or two come back as progress-update `thinking` blocks, whose text is empty at the default `display`. For visible notes, use `display: "updates"` (beta), or `between_tools`, which returns the summary text with no `display` field.
- Silent long turns: the harness can count consecutive tool steps without a user-facing message (the guide's example is five) and append a one-turn reminder such as "The user hasn't heard from you in a while: say in a few words what you're doing, then continue." Append reminders rather than inserting or deleting them, to keep the cache and preserved thinking intact.
- Exact text for the user mid-turn: declare a simple send-message tool in the first request so the `tools` list does not change later.
- Mid-turn user input: deliver it as a `text` block in the user message that carries the `tool_result` blocks, after the last `tool_result`. Keep harness notices in a separate mid-conversation system message after the user's words.
- Harness tolerance: accept case-mismatched tool names when the match is unambiguous, or return `is_error: true` stating the exact expected name.
- Long-context guidance: Not covered.

## Implications for skill files and agent instruction files
A compliant file (guide-derived checklist; items marked (inference) are my reading):
- States an effort level for each task type and says to re-sweep against evals rather than reuse Sonnet 5 settings.
- Does not discourage tool use, and tells the model to search for facts that may have changed.
- Includes a real-check verification rule for any file that involves code changes.
- States scope limits explicitly if unrequested additions are unwanted, and states whether to stop for check-ins (inference: this matters most at `low`/`medium`).
- Gives a clear stop-and-report instruction and, for open-ended work, an "ideas first" rule where relevant.
- For structured reasoning output, uses structured outputs or the "Think the problem through" line, and tells the caller to treat `max_tokens` as failed.
- Does not ask for reasoning in the response text.
- Does not ask the model to think less in the prompt; uses effort instead.
- Specifies update points for progress rather than "hold findings".
- Keeps user input out of tool results and separates harness notices from user text (inference: relevant when the file governs a harness).
- Does not use `between_tools` with `xhigh`/`max`, and does not set `max_tokens` for a no-thinking request.

A non-compliant file typically: discourages tools, asks for reasoning in output, asks the model to think less, relies on system-prompt effort control, has no verification rule for code changes, or says "hold all findings for the final response".

## Notable quotes
1. "Start at `high`, the default on the Claude API, unless your workload is agentic or latency-sensitive."
2. "Asking the model in the system prompt to think less doesn’t reliably reduce its thinking."
3. "Think the problem through before you answer."
4. "Never put user text inside a `tool_result` block. The model misreads that placement most often."
5. "On agentic coding tasks, Claude Sonnet 5.5 generally checks its work before it reports a change as done. At `low` effort, though, it sometimes reports a change as done without running a check that exercises it."
