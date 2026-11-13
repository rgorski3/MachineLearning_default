# youtube-to-ebook: model-specific rewrites

Source skill: `SKILL.md.bak` (identical to `skills/youtube-to-ebook/SKILL.md` on `origin/claude/youtube-to-ebook-skill-w2cerv`).
Outputs: `skill-haiku.md`, `skill-sonnet.md`, `skill-opus.md`, `skill-fable.md`. Each is a drop-in SKILL.md with the same frontmatter fields (`name`, `description`).

Every factual statement in the four files was checked against the scripts on that branch: `main.py`, `get_videos.py`, `get_transcripts.py`, `write_articles.py`, `send_email.py`, `video_tracker.py`, `dashboard.py`, `run_newsletter.sh`, both plists, `launch_dashboard.command`, `requirements.txt`, `.env.example` and `README.md`. The check was done by reading the code. No script was run, because that needs live API keys and would send email.

Guide sections are cited as *Guide → Section*, using the headings of the source guides in `scratchpad/guides/`.

## Changes common to all four versions

These come from the code, not from a guide, so they are the same in every version.

- **Factual corrections.** The original's wrong commands, files and behaviors are fixed. The full list is in "Errors and unverifiable claims in the original" below.
- **Pipeline description now matches `main.py`.** All versions describe the six steps, including the step that skips already-processed videos and the archive step. They also state the important failure behavior: if the email fails, nothing is archived or marked, so the next run processes and pays for the same videos again.
- **Setup is concrete.** "I'll guide you through" became numbered steps with real commands, ending with `python3 get_videos.py` as a cheap check before the paid `python3 main.py` run.
- **New reference material, all present in the code:** the five env vars and where each is used; the standalone script commands; the dashboard pages and three of its surprising behaviors; the hard-coded `/Users/bytedance/...` paths that must be replaced; the `logs/` folder that must exist; the f-string rules for editing the article prompt.
- **Named risky steps that need the user's go-ahead.** These are `main.py` / Generate, `send_email.py`, `launchctl bootstrap`, and editing `processed_videos.json`. All four guides say to keep your own confirmation step for risky or irreversible actions when you add keep-going text (*Haiku → Prevent early stopping*, *Sonnet → Steer initiative and scope*, *Opus → Unattended agentic runs*, *Fable → Finish the whole task*). The wording differs per model, as described below.
- **Description frontmatter rewritten** so it states what the skill actually does (channels → Claude articles → EPUB → Gmail) and when to use it. The `name` is unchanged.
- **Nothing added that a guide warns against.** No version tells the model to think more, think less, answer directly, or show its reasoning in the response (*Haiku → Use effort to control thinking*; *Sonnet → Calibrate effort*, *Safeguard refusals*; *Opus → Thinking instructions in chat system prompts*, *Safeguard refusals*; *Fable → Review prompts and harnesses designed for earlier models*, *Safeguard refusals*). The original had no such lines, so nothing had to be removed.
- **Left out on purpose in all versions:** search nudges and the current date (*Haiku → Accurate search results*, *Sonnet → Tool use in chat and knowledge work*). The skill does no web research, and the guides tie this text to products that give the model a search tool. The guides' sections on JSON output, mid-turn user messages, chatbot adherence, pasted text, frontend design, visual inputs and multi-agent time budgets are also left out, because they don't apply to this skill.

## Claude Haiku 5.5 (`skill-haiku.md`)

| Change | Why | Guide section |
|---|---|---|
| Kept the original's structure, section order, headings and pitfall code snippet. The only changes are factual fixes and two added paragraphs. | "Existing Claude Haiku 4.5 prompts should perform well without changes." The guide is a set of fixes for observed symptoms, not a rewrite recipe. | *Haiku → introduction* |
| Added the guide's early-stopping text, in its own wording ("Don't add new features, docs, or refactors…"). | Setup is a multi-step agent task carried in a long prompt, which is where the guide reports early stops and hand-backs, especially at `low` effort. | *Haiku → Prevent early stopping in long agent prompts* |
| Added the guide's verification paragraph, adapted to this project: run `get_videos.py` after a `CHANNELS` edit; install with `pip install -r requirements.txt`, not sudo; say plainly when the real check (`main.py` / `send_email.py`) wasn't run because it needs approval. | At `low` and `medium` effort the model "sometimes reports a code change as done without running a check". The guide's own example of installing dependencies is `pip install -r requirements.txt`, which is exactly this project. | *Haiku → Tell coding agents to verify their changes* |
| No progress-update instructions. | The Haiku guide has no progress-update section. | (absent from guide) |

**Deployment notes (API settings that can't go in a SKILL.md):**
- Effort: start at `medium` (the default, recommended "for most work, including agentic coding"). Use `high` if the model skips the approval list or other rules. At `low`, expect more early stops and skipped checks. (*Use effort to control thinking*)
- Don't send `budget_tokens`, which returns a 400. Leave room in `max_tokens` for thinking (up to 128,000). `thinking: {"type": "disabled"}` works only at `low`/`medium`/`high`. (*Use effort to control thinking*)
- Load the skill in new conversations. Resuming a stored conversation with a changed system prompt while sending thinking blocks back can return a 400. (*introduction note*)
- If reasoning-like text shows up in replies, use adaptive thinking at `medium`. (*Keep reasoning out of user-facing text*)

## Claude Sonnet 5.5 (`skill-sonnet.md`)

The reference body is the same as Haiku's, because the two guides agree on content. Only the Working Rules differ.

| Change | Why | Guide section |
|---|---|---|
| Keep-working paragraph plus the Sonnet wording of the scope paragraph ("Don't add features, tests, files, docs, or refactors…"), with project examples: no `channels.txt`, no CLI flags, no README rewrite. | At `low`/`medium` it checks in before multi-part work is done. Separately, it "tends to add tests, documentation, and small supporting files… at every effort level". The original skill invited exactly such additions (a `channels.txt` and a `--channels` flag that don't exist). | *Sonnet → Steer initiative and scope* (Carrying work through; Unrequested additions when coding) |
| "When the user asks for ideas or options… give them that and stop" (for example, other writing styles). | On open-ended requests it can start building when only ideas were wanted. The Customization section invites requests like "what styles could I use?". | *Sonnet → Steer initiative and scope* (Open-ended requests) |
| A one-line intent before the first command, and a recap at the end listing what ran, what it printed, what changed and what the user still needs to do. | "If you then want updates at predictable points… say so in the system prompt. The model follows instructions like this." Setup is human-in-the-loop work. | *Sonnet → User-facing progress updates* |
| Verification paragraph, same adaptation as Haiku. | At `low` effort it sometimes reports a change as done without a real check, for example skipping tests because dependencies aren't installed. | *Sonnet → Verification on coding tasks* |
| Left out the "don't start extra rounds of review" paragraph. | It addresses `xhigh`/`max` thoroughness, and routine work like this should run at `high` or below. | *Sonnet → Steer initiative and scope* (Thoroughness) |

**Deployment notes:**
- Effort: for multi-step tool use, start at `medium` for well-specified tasks and move to `high` for harder ones. Don't carry over Sonnet 5 values, because the levels were recalibrated. (*Calibrate effort*)
- `max_tokens`: leave room for thinking. For agentic coding, use 128,000 and stream. (*Calibrate effort*)
- Set `display: "updates"` (beta header `thinking-display-updates-2026-08-18`), or the intent and progress notes between tool calls come back as empty `thinking` blocks. (*User-facing progress updates*)
- If running without up-front thinking, use `between_tools`, which is accepted only at `high` or below. (*Running without up-front thinking*)

## Claude Opus 5.5 (`skill-opus.md`)

| Change | Why | Guide section |
|---|---|---|
| New "Before you act" section: read `get_videos.py`, `write_articles.py`, `.env` variable names, `processed_videos.json`, `newsletters/`, `logs/` and the LaunchAgents plist before changing anything, "including ones it doesn't mention". | Opus 5.5 "tends to get to work quickly, and on loosely specified tasks it helps to tell the model to look through the relevant sources before acting." The guide's sentence targets multi-app workflows. Applying it to a project's files is my inference. It fits this skill because the original pointed at the wrong place (`channels.txt`), and the facts live in files the user won't name. | *Opus → Explore context in multiapp workflows* |
| One-line intent before the first command and a short recap at the end. | Third lever: "if you want… a one-line statement of intent before the first tool call and a short recap at the end, say so in the system prompt." | *Opus → User-facing progress updates* |
| Specific "don't add" items: no `channels.txt`, no `--channels` flag. | The model "responds well to instructions that name specific patterns to avoid". General instructions mostly swap one default for another. That sentence is from the frontend section; applying the principle here is an inference. | *Opus → Frontend design defaults* |
| Leaner and restructured: risky steps up front, pitfalls turned into "Design decisions to keep" with function names instead of code snippets, and a merged troubleshooting list. | Existing Opus 5 prompts work, and Opus is strongest at multi-step work in a real repository. Since the skill now tells it to read the scripts, repeating code snippets adds nothing. (Inference from *Capabilities relevant to prompting* and the explore instruction above.) | *Opus → introduction*, *Capabilities relevant to prompting* |
| No keep-working, unattended-run, or verification paragraph. | The unattended-run paragraph is for "agents that run fully unattended". The guide says to "leave the addition out of human-in-the-loop applications", and setup is human-in-the-loop. (The launchd job runs Python, not Claude.) The Opus guide has no coding-verification section. | *Opus → Unattended agentic runs* |

**Deployment notes:**
- Effort: set it explicitly. Start at `medium` (the Opus 5.5 default; Opus 5 defaulted to `high`), and try `low`. Reserve `xhigh`/`max` for measured gains. (*Calibrate effort*)
- Thinking is always on, and `thinking: {"type": "disabled"}` returns a 400. Size `max_tokens` for thinking (128,000 worked well for long agentic turns). (*Calibrate effort*, *Prompts written for thinking disabled*)
- Set `display: "updates"` so the intent and progress notes are visible. (*User-facing progress updates*)
- If this skill is ever driven by an unattended loop, add the guide's unattended-run paragraph from the first request, and treat a text-only `end_turn` as a report rather than completion. (*Unattended agentic runs*)

## Claude Fable 5.2 (`skill-fable.md`)

| Change | Why | Guide section |
|---|---|---|
| "The user owns these scripts. Make the changes they ask for inside the existing structure…" The ownership sentence is deliberately left out. | Fable "pays close attention to who owns the code". Saying the code is its own led to consolidation, unrequested tests and docs, and changed behavior. The guide says to leave the sentence out "where the user owns the code and wants it changed as little as possible". | *Fable → Say who owns the code* |
| "Finishing work" block adapted from the guide: a definition of done for setup; ordinary obstacles named from this project (pip failure, "Channel not found", Supadata 429/timeout, missing `logs/`, `ModuleNotFoundError`) to diagnose and work through; user-only blockers (API keys, Gmail app password, approvals) to report while continuing with other work. | Fable "sometimes ends a turn before the work is done… stopping at an ordinary obstacle such as a timeout". The guide's text separates ordinary obstacles from deliberate blockers. | *Fable → Finish the whole task* |
| Approval list with a reason for each step, phrased as "Ask the user before…". No bare "never" rules. Added an explicit list of steps that need no approval. Asks for approvals that name the step ("Yes, run main.py now"). | "The model treats an instruction that contains 'never' as absolute"; "Name the steps that need no approval"; "be specific about the actions you want double-checked"; approvals should say exactly what is approved. | *Fable → Keep caution from piling up*, *Approvals* |
| The guide's "Signal intent" text, used nearly verbatim (opening line, brief updates, a recap that stands on its own). | Fable "might make its first tool call without saying what it's about to do". The guide says to use this text where a human is in the loop. | *Fable → Keep users informed during long turns* (Signal intent) |
| "Final report" paragraph, a short adaptation of "Communicating clearly": write for a low-context reader, result first, plain words, one name per thing. | The people who read the final report usually haven't seen the tool calls. The guide says to adjust the text to your readers. | *Fable → Make reports easy to follow* |
| Reference sections rewritten as connected prose, with bullets kept for separate items. No hard style limits (word counts, sentence-length rules). | Fable follows style rules closely, and hard limits make its prose choppy. "Use a paragraph of flowing prose where ideas connect, and keep bullets for lists of separate items." | *Fable → Review prompts and harnesses…* (Style rules), *Make reports easy to follow* |
| No delegation or subagent instructions, and no token budget text. | Standing delegation lines and token budgets shown to the model raise cost. | *Fable → Review prompts and harnesses designed for earlier models*, *Set expectations for subagents* |

**Deployment notes:**
- Effort: the default and recommended start is `high`. This is interactive setup work, for which the guide suggests `medium`, so test `medium`. The Finishing-work block matters most at `low`/`medium`. (*Calibrate effort*)
- Don't add "answer directly" lines. They don't reduce thinking at `medium` and above, so lower effort instead. (*Calibrate effort*)
- Set `display: "updates"` or `"summarized"` so progress notes are visible. A harness can also append the turn-scoped "The user hasn't heard from you in a while…" reminder. (*Keep users informed during long turns*)
- Load the skill in new conversations. A changed system prompt in a resumed conversation fails the prefix check with a 400. (*introduction note*, *Review prompts and harnesses…*)

## How the four versions differ

| Aspect | Haiku | Sonnet | Opus | Fable |
|---|---|---|---|---|
| Structure | Original layout kept | Same as Haiku | Leaner, reordered, no code snippets | Same content as Opus, written as prose |
| Explore project first | No | No | Yes | No (reading is listed as a step that needs no approval) |
| Keep-working text | Guide text | Guide text | Omitted (human-in-the-loop) | Adapted "Finishing work" with obstacle/blocker split |
| Scope limit | "No new features, docs, refactors" | "No features, tests, files, docs, refactors", with examples | Named items (`channels.txt`, `--channels`) | "User owns these scripts", minimal change |
| Ideas-first rule | No | Yes | No | No |
| Verification paragraph | Yes | Yes | No | No |
| Intent + recap | No | Yes | Yes | Yes (guide text) plus low-context report paragraph |
| Approvals | List of risky steps | List of risky steps | List of risky steps | Risky steps with reasons, plus a no-approval list; no "never" |
| Recommended effort (deployment) | `medium` | `medium`→`high` | `medium` | `high`, test `medium` |

## Errors and unverifiable claims in the original

| # | Original | What the code does | How it was handled |
|---|---|---|---|
| 1 | `python main.py --channels` edits the channel list | `main.py` takes no arguments; there is no argument parsing anywhere. | Removed. Channels are edited in `CHANNELS` in `get_videos.py` or on the dashboard's Channels page. All versions say there are no CLI flags. |
| 2 | `python dashboard.py` launches the dashboard | `dashboard.py` is a Streamlit app. `launch_dashboard.command` and the dashboard plist run `python3 -m streamlit run dashboard.py`. `streamlit` is not in `requirements.txt`. | Corrected the command and added `pip install streamlit`. |
| 3 | `channels.txt` holds the channel list | No such file. Channels are the `CHANNELS` list in `get_videos.py`, which `dashboard.py` edits with a regex. (`README.md` repeats the `channels.txt` claim.) | Corrected in all versions. README not changed, because it's outside this task. |
| 4 | Transcripts come from `youtube_transcript_api`; Pitfall 3 "use `ytt_api.fetch()`"; workflow box "Transcript API" | `get_transcripts.py` calls the Supadata API (`https://api.supadata.ai/v1/transcript`) with `SUPADATA_API_KEY`. `youtube-transcript-api` is in `requirements.txt` but no script imports it. | Pitfall 3 removed, and Supadata documented. Its error handling (404/401/429/timeout) replaces the old pitfall. |
| 5 | Requirements: YouTube key and Anthropic key only | `SUPADATA_API_KEY` is also required, and Gmail credentials are needed for a complete run. `.env.example` has no `SUPADATA_API_KEY` line. | All five variables listed. Setup tells the model to add the missing line. `.env.example` not edited (outside this directory). |
| 6 | Pitfall 4: 2-second delay between transcript requests | `time.sleep(1)`: 1 second, and not after the last video. | Corrected to one second. |
| 7 | Pitfall 6 plist: label `com.youtube.ebook`, runs `/usr/bin/python3 /path/to/main.py` | Shipped `com.youtube.newsletter.plist` runs `/bin/bash run_newsletter.sh`, Wednesday 07:00, with log paths and a working directory. `dashboard.py` reads and rewrites `~/Library/LaunchAgents/com.youtube.newsletter.plist` by that exact name. | Replaced with install steps for the shipped plist, and a note to keep its file name. |
| 8 | "GitHub Actions and cloud servers are blocked by YouTube for transcript fetching" | Can't be verified from the scripts. That reason applied to direct YouTube transcript fetching. The current code uses Supadata, whose comment says it "doesn't get blocked". | Claim dropped. The skills say only what is verifiable: the scheduling files shipped are macOS launchd only. |
| 9 | Email delivery is "Optional" | `main.py` always calls `send_newsletter()`. Without Gmail credentials the send fails, nothing is archived or marked processed, the EPUB stays in the project folder, and the next run repeats the paid calls. | Corrected. All versions say Gmail is needed for a complete run, and describe what a failed send leaves behind. |
| 10 | Writing style options: magazine, academic, casual blog, technical docs | There is one prompt and no presets. | List removed. The skills describe editing the single prompt and the f-string rules. |
| 11 | Python 3.8+ | No version is pinned anywhere in the project. | Unverifiable, so changed to "Python 3". |
| 12 | Pitfall 2 snippet: `part="contentDetails"` | Code uses `part="snippet,contentDetails"`. | Corrected (Haiku and Sonnet keep the snippet; Opus and Fable cite the calls). |
| 13 | Key Files tree lists 7 entries | It omitted `dashboard.py`, `video_tracker.py`, `run_newsletter.sh`, both plists, `launch_dashboard.command`, `requirements.txt`, `processed_videos.json`, `newsletters/`, `logs/`. | Full list given. |
| 14 | Example output: "original video links", "mobile-friendly styling" | The video URL is plain text, not a hyperlink. The CSS is a simple Georgia serif layout, and "mobile-friendly" can't be verified. | Reworded to what `create_epub()` produces. |
| 15 | (not mentioned) | `run_newsletter.sh`, both plists and `launch_dashboard.command` hard-code `/Users/bytedance/youtube-newsletter`. The dashboard plist hard-codes a Python 3.11 framework path. `logs/` must exist for the log redirects. | Added replacement and `mkdir -p logs` steps. |
| 16 | (not mentioned) | Bug: the `write_articles.py` self-test has no `description` key, so building the prompt raises `KeyError: 'description'` (found by reading the code, not by running it). | Documented in every version, with a pointer to test through `main.py`. Code not changed, because the rules forbid changing what the skill does. |
| 17 | (not mentioned) | The dashboard's "Clear All History" button only shows a warning. Its handle regex `@[\w]+` hides handles containing `.` or `-` and drops them on the next save. `/channel/UC…` URLs become unresolvable `@UC…` handles. | Documented as dashboard behaviors to work around. Code not changed. |
| 18 | (not mentioned) | The article model is hard-coded to `claude-sonnet-4-20250514` with `max_tokens=8000`. | Stated, with the README's own tip to raise `max_tokens` if articles are cut off. The model wasn't changed, because that would change the skill's output. |

## Where the summaries disagreed and the source guide was used

- *Sonnet, unrequested additions:* the Haiku-written summary ties them to `xhigh`/`max`. The source says "at every effort level, and more at higher effort". So the Sonnet scope paragraph is unconditional.
- *Scope wording:* the Haiku guide's text says "new features, docs, or refactors", and the Sonnet guide's says "features, tests, files, docs, or refactors". Each version uses its own guide's wording.
- *Opus, effort:* the Sonnet-written summary lists "Reserving `xhigh`/`max` by default" as an anti-pattern. The source says to reserve them for measured gains, and the deployment note follows the source.
- *Opus, exploring first:* the summaries quote the sentence without saying which section it's from. The source places it under multi-app workflows, so its use here is marked as an inference.
- *Fable, reports:* the Haiku-written summary says final reports "are often dense and use semicolons" as a trait. The source says the "Communicating clearly" text made them less dense. The adaptation follows the source.
