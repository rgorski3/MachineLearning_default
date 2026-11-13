# Claude Sonnet 5.5 prompting guide: summary (by Sonnet)
Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5 · fetched 2026-11-13

Scope note: this guide is about model-specific patterns (effort, initiative, thinking modes, JSON output, progress updates, tool use, verification, refusals). It is API/harness-heavy. It says existing Sonnet 5 prompts "should perform well without changes" and that general techniques live in the separate "Prompting best practices" guide. It is not a general guide to writing skill files.

## TL;DR
- Effort is the main control for thinking, quality, latency and cost, and its levels are recalibrated versus Sonnet 5. Re-sweep against your own evals instead of carrying the old setting over. To get less thinking, lower effort; telling the model in the prompt to think less "doesn't reliably reduce its thinking".
- Behavior at the edges is steered with short system-prompt paragraphs. At low and medium effort it may stop to check in on long agentic tasks. At high effort, xhigh and max it may do more than asked (extra tests/docs, review rounds, reviewer subagents). The guide supplies copy-paste paragraphs for each.
- Remove prompt language that discourages tool use ("only use tools when strictly necessary", "minimize tool calls") and instructions like "hold all findings for the final response". Add targeted instructions instead (search for changeable specifics, run a real check before reporting done, update at set points).
- For JSON answers to multi-step reasoning tasks, the model often skips thinking. Use adaptive thinking, add "Think the problem through before you answer.", or use xhigh. Do not use `between_tools` for tools-free reasoning requests.
- Harness text placed right after tool results (countdowns, per-step instructions, user text inside `tool_result`) can be misread as prompt injection. Deliver user input as a user turn.

## What's distinctive about this model
- Effort: levels are recalibrated; the same level does not give the same thinking as on Sonnet 5. Default on the Claude API is `high`.
- Low effort: keeps thinking short, can skip verifying a change (for example skipping tests because dependencies are not installed).
- Low and medium effort on long agentic tasks: more likely to pause to confirm a plan, ask a question it could answer itself, or stop after one part of a multipart task.
- Unrequested additions: tends to add tests, documentation and small supporting files that fit repo conventions, at every effort level and more at higher effort. The requested change itself stays close to what was asked.
- xhigh and max: especially thorough; after finishing it can start its own review and verification rounds (sometimes with subagents if the harness provides them) and make related fixes it noticed.
- Open-ended requests ("show me what you can do with this"): can start building a presentation, report or video when only ideas were wanted.
- Reasoning plus JSON: often answers without thinking first, especially at low and medium effort, which lowers accuracy on multi-step tasks (totaling figures, applying a rule, ranking).
- Between tool calls it writes user-facing notes about what it found and what it is doing next. Notes longer than a sentence or two come back as progress-update `thinking` blocks, which are empty at default display, so a client rendering only `text` can look silent.
- Chat and knowledge work: sometimes answers from training knowledge when a search would catch changed details (what is allowed, required, charged).
- Trained to resist indirect prompt injection, and sometimes treats a genuine mid-task user message as a possible injection.
- Occasionally calls a declared tool with wrong letter case (`bash` for `Bash`) or a slightly different parameter name.
- Reads dense charts and technical drawings markedly better with crop/zoom/code tools.
- Runs safety classifiers that can return `stop_reason: "refusal"` (categories: cyber, bio, frontier_llm, reasoning_extraction, general_harms).
- Sibling comparison: for the hardest long-horizon work, "an Opus model is the better choice."

## Recommended practices
- Effort starting points: `high` by default; for agentic coding and multistep tool use start at `medium` for well-specified tasks and `high` for harder or longer ones; for chat or latency-sensitive work start at `medium` or `low`. Reserve `xhigh` and `max` for work where a quality gain is measured.
- `max_tokens`: leave room for thinking plus reply (thinking counts even if not returned). For agentic coding set 128,000 and stream.
- Change effort mid-conversation with per-message effort (beta) rather than changing top-level `effort`, which invalidates the prompt cache.
- To keep it working at low/medium effort (system prompt, verbatim):
  "Keep working until everything the user asked for is done, and only stop to ask when you can't go on without the user or before a risky step."
- To limit scope: "When the work the user asked for is done and checked, stop and report. Don't add features, tests, files, docs, or refactors that weren't asked for. If you think one would help, mention it at the end instead of doing it." (The second paragraph alone is enough to limit additions; at xhigh/max it also makes changes smaller.)
- To suppress self-started review at xhigh/max: "When the work the user asked for is done and its checks pass, stop and report. Don't start extra rounds of review or hardening on your own, and don't launch reviewer subagents unless the user asked for a review. If you think a deeper review is worth doing, say so at the end." (At max effort this cut session cost about a third with no quality change; it reduces but does not eliminate self-started review.)
- For ideas or plans: "When the user asks for ideas, options, or a plan, give them that and stop. Don't start building or changing anything until they say to go ahead."
- Verification (add if changes are reported done without test/build output): instruct it to run a real check that exercises the change (tests, type-checker, build, or the changed command); a syntax-only check or a check that failed to start does not count; install only declared dependencies with the project's package manager and lockfile, never via sudo or the system package manager unless told not to; if no real check can run, say which one was not run and why. At low effort this makes skipped or superficial checks rare.
- JSON with reasoning: with adaptive thinking, end the system prompt with "Think the problem through before you answer." (at high effort this brings accuracy close to xhigh). Or use xhigh. Prefer structured outputs. Without structured outputs, parse the last JSON value in the `text` blocks (not first `{` to last `}`), validate fields, retry once. Treat `stop_reason: "max_tokens"` as failed.
- Search: "Use the search tool to check specifics that may have changed since your training, such as what is allowed, required, or charged, even when you feel confident. For researched work such as a report or a comparison, gather current sources rather than writing from your training knowledge."
- Progress updates: if you want updates at predictable points (for example a line before the first tool call and a short recap at the end), say so in the system prompt; "The model follows instructions like this."
- Mid-turn user input: append the user's words as a text block in the user message that carries the `tool_result` blocks, after the last one; keep harness notices in a separate system message after the user's words.

## Anti-patterns / things to stop doing
- Carrying over the Sonnet 5 effort setting unchanged.
- Asking in the system prompt to think less (unreliable); the fix is lower effort.
- Prompt language that discourages tool use ("only use tools when strictly necessary", "minimize tool calls").
- Older instructions such as "hold all findings for the final response" if you want progress updates.
- Instructions telling the model not to think when using `between_tools`: they make it more likely to write internal XML tags in visible output (remove them).
- Instructions asking the model to include its reasoning in the response text: they invite `reasoning_extraction` refusals. Read summarized thinking blocks instead; short explanations of the answer or summaries of actions are still fine.
- Using `between_tools` for tools-free reasoning tasks (it means no thinking before answering; the "think first" line has no effect).
- Putting user text inside a `tool_result` block (most common misread).
- Token or budget countdowns after every tool result in interactive sessions; per-step injected instructions after tool results; frequent harness reminders (send fewer; stop after the second or third if the turn stays quiet).
- Mixing a harness notice and the user's words in the same block.
- Assuming the first content block is text; not passing `thinking` blocks back unchanged.
- Treating wrong-case tool names or parameter names as fatal.
- Splitting a reasoning-JSON task into two requests (accurate but very high cost and latency).
- Parsing a whole response as JSON when JSON was requested in-prompt without structured outputs.

## Agentic, tool-use and long-context guidance
- Agentic coding: start at `medium`/`high`; `max_tokens` 128,000 with streaming; add the carry-through, scope and verification paragraphs above as needed. The carry-through prompt makes low/medium sessions run longer and cost more, and "doesn't replace your own rules about risky or irreversible actions. Keep those rules in your system prompt."
- Silent long turns: set `display: "updates"` (beta) or use `between_tools` (summary text returned; no other fields accepted with it). Give the model a simple message-the-user tool for exact text (code snippet, question), declared in the first request so `tools` does not change; tell it to use the tool only for such content. Optional harness reminder after about five consecutive silent tool steps, sent as a turn-scoped system message: "The user hasn't heard from you in a while: say in a few words what you're doing, then continue." Leave reminders in `messages` on later requests to preserve cache and preserved thinking.
- Tool-call tolerance: accept unambiguous case-mismatched names, or return `tool_result` with `is_error: true` stating the exact expected name; the model usually corrects itself.
- Visual inputs: provide crop/zoom/code tools. For charts they help at every effort level (tools at high beat no tools at max, at a fraction of cost); for technical drawings they help only from high up, most at xhigh/max.
- Thinking-mode constraints: `between_tools` is accepted only at `high` or below, takes no `display`, `budget_tokens` or `block_binding`, and cannot be combined with per-message effort changes (400 errors). Per-message effort needs adaptive thinking.
- Refusals: server-side fallback (beta) retries `cyber` and `frontier_llm` declines on Sonnet 5, not bio, reasoning_extraction or general_harms. Verification programs exist for cyber and life sciences work.
- Long-context guidance: Not covered.

## Implications for skill files and agent instruction files
Checklist (items marked "(inference)" extend the guide to file-writing; the rest apply it directly).
A compliant file:
- Uses short, direct, outcome-stating paragraphs for steering, like the guide's own examples, rather than long procedural scaffolding (inference).
- States when to stop and when to ask: keep going until done, ask only if blocked or before a risky step.
- States scope explicitly: whether unrequested tests/docs/refactors/files are wanted, and to mention rather than do optional extras.
- Keeps its own rules for risky or irreversible actions explicit; does not rely on carry-through wording to cover them.
- Says what counts as a real verification (tests, type-check, build, run the changed command), what does not (syntax-only, failed-to-start), how to handle missing dependencies, and to report an unrun check honestly.
- Says that ideas/options/plans requests get a plan and a stop, not a build.
- Where it covers research or support work with a search tool, says to verify changeable specifics even when confident.
- Says what progress updates are wanted and when, if any.
- Sets effort explicitly per task type and does not assume Sonnet 5 effort values carry over (inference: applies to files that name an effort level).
- Where JSON output depends on reasoning, specifies adaptive thinking/"Think the problem through before you answer." or higher effort, and a robust parser.
A compliant file avoids:
- Tool-minimizing language and "hold everything for the final response" language.
- "Don't think" / "think less" instructions.
- Requests to write out reasoning in the response.
- Harness-injected countdowns or per-step notices after tool results.
- Both over-steering (stop and ask constantly) and carry-through wording with no risk guardrails.
- Re-stating the whole old prompt for Sonnet 5 as if changes were required (guide: existing Sonnet 5 prompts mostly work; add only the paragraph that fixes an observed behavior) (inference).

## Notable quotes
- "Asking the model in the system prompt to think less doesn't reliably reduce its thinking."
- "Keep working until everything the user asked for is done, and only stop to ask when you can't go on without the user or before a risky step."
- "Use the search tool to check specifics that may have changed since your training, such as what is allowed, required, or charged, even when you feel confident."
- "Think the problem through before you answer."
- "Never put user text inside a `tool_result` block."
