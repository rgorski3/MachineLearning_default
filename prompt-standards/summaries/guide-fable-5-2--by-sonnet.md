# Claude Fable 5.2 prompting guide: summary (by Sonnet)
Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-2 · fetched 2026-11-13

(The guide covers Claude Fable 5.2 and, by its description, Claude Mythos 5.2, which has the same capabilities but is offered only to organizations verified via the Cyber / Life Sciences programs.)

## TL;DR
- Fable 5.2 follows instructions closely and stops early on long tasks at low and medium effort. Add a "Finishing work" block for agentic work, and keep your own confirmation step for risky or irreversible actions.
- Start at `effort: high` and sweep against your own evals. Do not carry over settings from Fable 5.1 or Opus 5.5. "Think less" instructions no longer work; lower the effort level instead.
- Stated defaults and wording have big behavioral effects. One sentence of code ownership changes refactoring behavior. An absolute "never" is treated as absolute. A vague approval rule makes it hold back routine work.
- Audit inherited prompts, skills, agent files, memory and notes. Remove standing delegate instructions, token budgets shown in the prompt, stale cautions, think-less lines, and rigid style rules.
- Make final reports self-contained for a low-context reader. Add the "Communicating clearly" text if reports and handoffs matter.

## What's distinctive about this model
- **Early stopping.** On long multistep tasks it sometimes ends a turn by listing what is left, offering to continue, or stopping at an ordinary obstacle (e.g. a timeout). More frequent at `low` and `medium`.
- **Always thinks at `high` and above.** It thinks before almost every reply, even a greeting. In Anthropic's testing, an instruction to answer directly without deliberating did not stop thinking at `medium` and above. (Fable 5.1 usually obeyed such a line.)
- **Effort controls** thinking, number of tool calls, turn time and tokens. It has little effect on reply length; use prompting for length and tone. At `low` it does better than past models on coding evals and scored close to Fable 5.1 at higher effort. Most of the capability gain comes at `high`; above `high` gains are uneven.
- **Attends closely to code ownership.** Without an ownership statement, or if the code is framed as someone else's, it adds to existing structure even when that duplicates code. If told the code is its own, it consolidates duplicates and sometimes makes unrequested related fixes.
- **Quiet during long tool-call runs.** It may make its first tool call without saying what it is about to do. Its between-tool updates arrive as progress-update `thinking` blocks whose text is empty at the default `display: "omitted"`, so a text-only client looks silent.
- **Delegation depends on harness.** It rarely starts subagents with a minimal subagent tool and short tasks, but delegates readily where prompts invite it. At `xhigh` and `max` it used subagents to review its own work (Fable 5.1 did not). Early testers liked it as planner/orchestrator for other models.
- **Follows rules literally.** It treats instructions containing "never" as absolute and follows style rules closely (a hard limit like "one instruction per sentence, in 20 words or fewer" makes prose choppy).
- **Writing:** testers rated it above Fable 5.1 and similar in clarity to Opus 5.5, but final reports often assume context the reader lacks.
- **Preserved thinking:** a request that fails the prefix check (changed `system`, `tools` or an earlier message) gets a 400 error.

## Recommended practices
- **Effort:** set `high` explicitly, then run evals at `medium` and `xhigh`. `low` for quick back-and-forth, well-specified tasks and high-volume subagents; `medium` for interactive coding, chat and everyday knowledge work; `high` for hard coding, research, agentic work and long unattended runs; `xhigh` only where a gain is measured; `max` for measuring the ceiling or rare open-ended problems. Move down to `medium` wherever evals hold. Mix levels in one workflow (plan at `medium`, run long unattended part higher), using per-message effort (beta) to keep the cache.
- **Finish the whole task (agentic work at low/medium).** Add a block headed `# Finishing work`, opening: "Please avoid stopping while you still owe work the user asked for. Be persistent and do not stop unless needed." It also says: carry on with work that does not depend on the user's answer; if you notice yourself inviting the user to redirect you or offering to wait, proceed instead; diagnose errors, timeouts, locked files, empty results and failing tools and work through them; leave deliberate blockers (a file marked not to be touched, withheld access, a safety guardrail) alone, say what you found, and look for another safe path; it "does not override the need for confirmation on risky or destructive actions." Measured effect at `medium`: about half as many sessions ended with continuable unfinished work, about 30% more output tokens; it removed most permission-asking or offer-to-continue endings but not "list what remains and stop" endings.
- **Approvals.** Reply with specifics, not "go" or a thumbs-up: "Yes, run the migration on the staging database". In system-prompt approval rules, name exact actions. A rule listing "sending messages to people outside this conversation" made the model hold back about half of routine requests to colleagues in no-approver simulations.
- **Ownership.** To get restructuring: "You are the owner and long-term maintainer of this codebase, and its design is up to you." Consolidation went from under 10% of runs to about 75%, with output tokens up about 70-80%, unrequested tests/docs, and sometimes changed existing behavior. Its closing PR summary often omitted small changes, so review the actual diff. Omit the sentence where the user owns the code and wants minimal change.
- **Progress visibility.** Set `display: "updates"` (short summary of each update) or `"summarized"` (updates plus summarized reasoning) and render non-empty `thinking` blocks between tool calls. Options, in order of strength:
  - Harness reminder, as a turn-scoped system message after several consecutive silent tool steps (count only with display `updates`/`summarized`): "The user hasn't heard from you in a while: say in a few words what you're doing, then continue." It roughly halved the longest silent stretch with no noticeable token or time increase. Leave each reminder in `messages` on later requests.
  - Prompt text: "Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own (what you found, what you did, and what's next)..." Every run wrote a pre-tool message; about 20% more output tokens. Use where a human is in the loop.
  - Fallback sentence: "Every few tool calls, give the user a one-line update on where things stand."
  - Expect several tool calls between updates even with all three. For work where nobody needs frequent updates, Anthropic recommends this model for long asynchronous tasks.
- **Subagents.** Change the subagent tool description only if it starts more than needed. State what a subagent adds to session cost, when delegation is and is not worth it, and how to brief a subagent. A longer cost-focused description roughly halved subagent count at `high`. Check which model subagents default to.
- **Reports.** For final reports/handoffs add a `# Communicating clearly` block: write for an intelligent, low-context reader who has not seen tool calls or files; self-contained messages; key points in the first few sentences; progressive disclosure (no undefined references early); plain words; no self-invented terms or abbreviations; mostly ASD-STE100 Simplified Technical English (short sentences, one idea each, active voice, same word for same thing); flowing prose where ideas connect, bullets only for separate items; omit unnecessary detail. It made reports less dense, with fewer semicolons and shorter sentences. Adjust for length or required details.
- **Mid-conversation changes.** Make prompt changes in new conversations. To change instructions mid-conversation, put them in the newest turn or append a mid-conversation system message.

## Anti-patterns / things to stop doing
- Telling it to "answer directly without deliberating" or otherwise think less. Lower the effort level instead.
- Carrying over effort settings from Fable 5.1 / Opus 5.5.
- Standing "delegate" instructions in system prompts, skills and agent files (remove if it over-delegates).
- Showing the model a token budget in the prompt (removing a large one lowered cost at equal performance at `high`/`xhigh`; this is about prompt text, not API task budgets, which were not tested).
- Harness code that edits the system prompt, tools or earlier turns between requests (invalidates thinking blocks; with `drop_block` the model re-thinks and uses more output tokens). Keep history append-only except compaction / context editing.
- Opus 5.5's two "treat that earlier answer as done" chat-prompt sentences: they reduced thinking on follow-ups and made the model far less likely to flag mistakes in its earlier answers. Leave out where self-correction matters.
- Rigid style rules (e.g. word-count-per-sentence limits).
- Absolute "never" rules. Write "ask for approval before deleting files", not "never delete files", to keep the model responsible.
- Vague permissions like "act more freely"; list the routine steps that need no approval.
- Over-broad stopping conditions such as "stop and review at the first error".
- Accumulated memory, notes and instruction files with stale cautions (e.g. "never retry"), some possibly model-written. Delete those whose reason no longer holds. A stuck system may be restarted with trimmed memory.
- Sending a bare "continue" or "go".
- Instructions to write reasoning into the response (invites `reasoning_extraction` refusals).
- Rule that limits scope does not cancel the ownership sentence ("No drive-by refactors, renames or cleanups" earlier in the prompt still left consolidation in more than half of runs).

## Agentic, tool-use and long-context guidance
- Agentic effort guidance and the "Finishing work" block are above. Keep your own confirmation step for risky or irreversible actions, since the block makes the model ask less.
- Multi-agent setups: caution accumulating through memory/notes/instruction files was seen mostly when several agents worked together.
- Preserved thinking: changing `system`, `tools` or an earlier message can fail the prefix check (400 on the Messages API; the Batches API drops the blocks). Use context editing or compaction to shorten history. `thinking.block_binding.prefix_mismatch_behavior: "drop_block"` (beta) is available but measure token cost first.
- Per-message effort (beta) changes effort without breaking the cache. Changing top-level `effort` can invalidate cached messages.
- Safeguard refusals: `stop_reason: "refusal"` with `stop_details.category` of `cyber`, `bio`, `frontier_llm`, `reasoning_extraction`, `general_harms` or `null`. Benign security and life-sciences work can trigger `cyber` / `bio`; the guide points to the Cyber Verification and Life Sciences Verification programs. To see reasoning, set `display: "summarized"` and read the thinking blocks.
- Long-context handling and tool-description design beyond the subagent tool: Not covered.

## Implications for skill files and agent instruction files (checklist)
A compliant file:
- Does not contain "think less / answer directly without deliberating" lines; effort is set in config instead.
- Contains no standing "always delegate to subagents" rules unless delegation is wanted; if it does mention subagents, it says what they cost and when they are worth it, and how to brief them.
- States code ownership deliberately: includes the ownership sentence if restructuring is wanted; omits it (and states scope limits) if minimal change is wanted. (inference: scope-limit rules alone are not a reliable brake, per the guide's test.)
- Names exactly which actions need approval and which routine actions do not; uses "ask before X" with scope and reason rather than bare "never X".
- Avoids blanket stop-on-first-error or stop-and-review conditions; says which obstacles to work through and which blockers are deliberate.
- Has no stale cautions, outdated memory or notes, and no token budget text.
- For agentic use at low/medium effort, includes (adapted) "Finishing work" text.
- For user-facing long runs, includes opening-line / periodic-update / standalone-recap instructions, or the one-line "every few tool calls" sentence.
- For reports and handoffs, includes the "Communicating clearly" text and avoids hard sentence-length style rules.
- Does not ask the model to write out its reasoning in the response.
- Does not rely on editing earlier conversation state; changes go in new conversations or the newest turn.
- Is tested one behavior at a time, measuring effect and token cost (guide's "Where to start" step 4).

An auditor should flag: think-less lines; delegation mandates; absolute "never" rules; vague approval rules; rigid prose-style limits; old-model "treat that answer as done" lines; token budgets in prompt text; cautions with no stated reason or scope.

## Notable quotes
- "Please avoid stopping while you still owe work the user asked for. Be persistent and do not stop unless needed."
- "You are the owner and long-term maintainer of this codebase, and its design is up to you."
- "The model treats an instruction that contains “never” as absolute."
- "Every few tool calls, give the user a one-line update on where things stand."
- "Write for an intelligent but low-context reader."
