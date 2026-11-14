# Overnight report: writing instructions for Claude 5.x models

**Date:** 2026-11-13. **Branch:** `ccr-ee583616-7bgump` of `MachineLearning_default`. Everything is in `prompt-standards/`.

## TL;DR

- **8 guide summaries done.** Haiku and Sonnet each summarized all four guides: Haiku 5.5, Sonnet 5.5, Opus 5.5 and Fable 5.2. Each summary was checked against a fresh copy of its source guide.
- **The skill was rewritten 4 times:** `skill-haiku.md`, `skill-sonnet.md`, `skill-opus.md` and `skill-fable.md`, for youtube-to-ebook. The rewrite also found **18 errors in the original SKILL.md**, for example the wrong transcript provider and a CLI flag and a `channels.txt` file that don't exist. It also found **3 real bugs in the scripts**.
- **No AGENT.md, AGENTS.md or CLAUDE.md file exists on any branch of either repo.** Haiku inventoried and graded 105 agent prompts and skill files. Exactly one file qualified for re-authoring: `/capture`. Opus rewrote it and the original is saved as `capture.md.bak`.
- **Nothing was changed on your working branches.** I can only push to `ccr-ee583616-7bgump`, so the new versions sit next to their `.bak` originals in `prompt-standards/`. You choose what to copy over.

## What the guides say, condensed

The guides are short sets of fixes for specific behaviors each model shows. They are not general prompting style guides. Every guide repeats some themes and each model also has its own.

**Across all four models**
1. **Effort is the thinking control.** Prose such as "think hard", "think less" or "answer without deliberating" is out. So is `budget_tokens`, which returns a 400 on Haiku 5.5. Depth comes from the API effort setting.
2. **Don't ask for reasoning in the reply.** Ask for a short explanation or a summary of actions instead.
3. **Keep going, with named stop points.** Agent prompts should say to keep working until the job is done, and name the only acceptable stops: a blocker only the user can clear, or a risky step.
4. **State the scope.** Say whether unrequested tests, docs, files or refactors are wanted. Extra ideas go at the end as suggestions.
5. **Require a real verification step.** For code, that means running the tests, the build or the changed command. A syntax check doesn't count.
6. **Input hygiene.** User text never goes inside a `tool_result`. Don't edit the system prompt mid-session.
7. **Keep an explicit confirmation step for risky actions.** Approvals should name the specific action.

**Per model**

| | Haiku 5.5 | Sonnet 5.5 | Opus 5.5 | Fable 5.2 |
|---|---|---|---|---|
| Default effort | medium | high (recalibrated) | medium | high |
| Main point of the guide | Older 4.5 prompts mostly carry over. Paste-in fixes for under-searching, early stopping and skipped verification | Limits on initiative and scope. Thoroughness rises at xhigh/max | Unattended loops: a text-only `end_turn` is a progress report, not completion | Follows prompts literally. Stops early at low/medium |
| Add | Today's date when search is available. "JSON applies to your final answer only" | "Think the problem through before you answer" for JSON. When asked for ideas, give them and stop | Wrap pasted text in tags with an ID. Name the specific early stops to avoid | A "Finishing work" block. A sentence saying who owns the code. A no-approval list |
| Remove | Blanket "always search" rules | Tool-minimizing wording | "Think carefully" in chat prompts. Vague "avoid a generic AI look" | "Never" rules without reasons. Pushes to delegate. Token budgets. Stale cautions |

The full checklist (13 items for all models plus model-specific items) is in `inventory.md` §1.

## Deliverables

| Path | What |
|---|---|
| `summaries/guide-<model>--by-{haiku,sonnet}.md` | The 8 summaries |
| `skills/youtube-to-ebook/skill-{haiku,sonnet,opus,fable}.md` | Drop-in SKILL.md, one per model |
| `skills/youtube-to-ebook/CHANGES.md` | Per-model changes with guide citations, a table comparing the versions, the 18 corrections, and deployment notes on effort, `max_tokens` and thinking |
| `skills/youtube-to-ebook/SKILL.md.bak` | The original skill |
| `inventory.md`, `reauthor-targets.txt` | Haiku's inventory and grading of every agent prompt and skill file |
| `reauthored/MachineLearning_default/claude__simulation-life-hacks-soslrp/.claude/commands/capture.md` | Re-authored `/capture` |
| `…/capture.md.bak`, `…/CHANGES.md` | The original command, and what changed and why |

## How the four skill versions differ

The versions differ only where the guides differ.
- **Haiku** keeps the original layout and adds the guide's keep-working and verification paragraphs.
- **Sonnet** has the same body with a stricter scope rule, plus the "give ideas and stop" rule and an intent line and recap.
- **Opus** is the leanest. It says to read the project before acting and leaves out the keep-working text, because that guide limits it to unattended runs and setup has a person in the loop.
- **Fable** is prose. It states that you own the scripts and want minimal change, separates obstacles to work through from blockers to report, gives a reason for every approval, and doesn't use "never".

All four came out 2.6–3.4× longer than the original (1,370–1,820 words against 533). Most of the growth is corrected setup facts, not style text.

## Decisions I made while you were asleep

1. **Which skill.** You said "1 Skill file" without naming it. I chose `youtube-to-ebook`, because it is the only skill you wrote yourself. The others are vendored from mattpocock/skills.
2. **What counts as an AGENT.md.** No file with that name, or AGENTS.md or CLAUDE.md, exists anywhere. Haiku searched for everything that does the same job: `.claude/commands`, agent briefs, CrewAI `agents.yaml`, and prompts embedded in code.
3. **What gets re-authored.** Only agent instruction files that you wrote. I did not rewrite:
   - the 35 vendored mattpocock skills, all graded Partially (forking them would break `skills-lock.json` updates);
   - the forked upstream projects in ai-engineering-hub;
   - CrewAI configs that target non-Claude models.
4. **Where the output goes.** The files live on other branches, and I'm only allowed to push to this one. So the outputs sit side by side with their `.bak` originals in `prompt-standards/`, and the source branches are untouched.

## Findings worth your attention

**youtube-to-ebook, where the original SKILL.md disagreed with the code.** The full list of 18 is in CHANGES.md.
- Transcripts come from **Supadata** (`SUPADATA_API_KEY`), not youtube-transcript-api. The key was **missing from `.env.example`** (fixed 2026-11-14, see below).
- Channels come from the `CHANNELS` list in `get_videos.py`. There is no `--channels` flag and no `channels.txt`.
- Email is not optional in practice. If the Gmail send fails, nothing is marked processed, so the next run pays Claude and Supadata again for the same videos.
- `run_newsletter.sh`, both plists and `launch_dashboard.command` hard-code `/Users/bytedance/youtube-newsletter`.
- `write_articles.py` still calls **`claude-sonnet-4-20250514`**. That is a pre-5.x model and the best follow-up candidate.

**youtube-to-ebook, bugs in the scripts. All fixed on 2026-11-14.** The fixes are on this branch at `skills/youtube-to-ebook/`, with one commit per fix after merging the skill branch in:
- The `write_articles.py` self-test raised `KeyError: 'description'`, because the test dict had no description key. The dict now includes one.
- The dashboard's "Clear All History" button only showed a warning and deleted nothing. It now asks for confirmation, then empties `processed_videos.json`. Cancel leaves the history alone.
- The dashboard's handle regex `@[\w]+` dropped handles that contain `.` or `-`. This was worse than it looked: the next add or remove on the Channels page rewrote `get_videos.py` without them, which deleted those channels. Pasting `youtube.com/@some.name` also added `@some`. Both patterns now allow `.` and `-`.
- `SUPADATA_API_KEY` is now in `.env.example`, with the placeholder value the script treats as unset.

A test script reproduced all four problems on the original code, failing 5 of 5 checks, and passes 5 of 5 on the fixed code. It drives the real dashboard with Streamlit's app tester, including the Cancel path, and stubs the Claude client so it makes no API call. The four skill versions and `CHANGES.md` were updated to match. Still open: `streamlit` is missing from `requirements.txt`, the plists and scripts hard-code `/Users/bytedance`, and a `youtube.com/channel/UC...` URL turns into `@UC...`, which the API can't find.

**`/capture`, what changed.**
- It now defines when a run is done.
- It no longer attempts an empty commit when there are no entries.
- It puts the transcript inside `<transcript>` tags, so a spoken "Action: …" goes on your checklist instead of being carried out.
- It states the commit is pre-approved and needs an explicit request before pushing.
- The frontmatter is byte-identical to the original.

**Summary quality.** Where two summaries disagreed, Opus checked the source guide. The Haiku-written summaries were wrong twice and the Sonnet-written ones once. Example: Haiku tied Sonnet's unrequested additions to xhigh/max only, while the guide says they happen at every effort level. The Sonnet summaries were also longer and kept more of the guides' exact wording. That is a small sample, so treat it as a leaning, not a verdict.

**A problem in my pipeline, caught in time.** The 8 summarizers downloaded into shared temp files, and one agent's copy of the Fable guide was overwritten by the Sonnet guide partway through. That agent noticed and re-fetched. I then checked every summary against a fresh copy of its source, and all 8 were on the correct guide. Later phases used separate files.

## Suggested next steps

1. Pick a skill version for the model you actually run (probably Opus or Sonnet). Copy it over `skills/youtube-to-ebook/SKILL.md` on `claude/youtube-to-ebook-skill-w2cerv` and keep the `.bak`.
2. Copy the new `capture.md` onto `claude/simulation-life-hacks-soslrp`.
3. ~~Add `SUPADATA_API_KEY` to `.env.example` and fix the 3 script bugs.~~ Done 2026-11-14. To apply the fixes to the skill branch: `git checkout claude/youtube-to-ebook-skill-w2cerv && git cherry-pick 3ba42bc c11b4b3 6af1bef 9c3ca9e`.
4. Optional: update the prompt and model in `write_articles.py` to 5.x, and run a model-specific pass over the vendored skills you actually use.

## Run log

| Phase | Model / effort | Agents | Wall time |
|---|---|---|---|
| Summaries | Haiku high ×4, Sonnet high ×4 | 8, in parallel | 25–49 s each |
| Skill rewrite | Opus 5.5 medium | 1 | ~7 min |
| Inventory | Haiku high | 1 | ~9.5 min |
| Re-author `/capture` | Opus 5.5 high | 1 | ~2.5 min |
