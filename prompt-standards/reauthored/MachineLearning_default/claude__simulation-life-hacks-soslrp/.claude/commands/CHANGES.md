# capture.md: re-authoring notes

Source: `.claude/commands/capture.md` on `origin/claude/simulation-life-hacks-soslrp` (original kept as `capture.md.bak`).
Standard: `prompt-standards/inventory.md` section 1, plus the four 5.x guides. The inventory graded the file Partially (gaps 5 and 6). A full re-check found two more gaps that matter in practice: no rule for a run with zero glitches, and no statement that the commit is pre-approved, which conflicts with Claude Code's default of "commit only when the user asks".

Unchanged: frontmatter (`description`, `allowed-tools`), inputs (`$ARGUMENTS`), the output file, the entry format, the filing rule (newest first, under the marker), the commit message, "no push", the chat output, and the owner's voice and self-description.

## Changes

| # | Change | Checklist item / guide section |
|---|---|---|
| 1 | New "When you're done" section. It defines done (glitches filed and committed, or there were none, and the step 7 output written), says to do all the steps in one pass without check-ins, says to retry ordinary snags (re-read and retry a failed Edit), and names the two cases not to work around: a missing `<!-- CAPTURE-INSERT -->` marker and a failed commit. In both cases the actions and notes still go out, plus one line saying what failed. | 5; Fable 5.2 "Finishing work" (ordinary obstacles vs. deliberate blockers); Sonnet/Haiku 5.5 keep-working paragraph; F1 |
| 2 | Scope line: the only change is new entries under the marker plus one commit, with no other edits, files or commits. Other ideas get one line at the end and are not acted on. | 6; Sonnet/Haiku 5.5 scope paragraph ("mention it at the end instead of doing it") |
| 3 | Step 2: "Don't stop to ask about an item's type. Pick the likeliest type, keep going, and flag the guess in step 7." This closes the inventory's note that step 2 invited guesses with no rule for asking, and ties step 2 to the existing step 7 flag. | 5, 6 |
| 4 | Zero-glitch case: "If there are no glitches, skip this step and step 6." The original always committed, so a run with only actions or notes would have tried an empty commit. | 5 (defined end state) |
| 5 | Step 6: "Running `/capture` is his request for this commit, so don't ask first." This names the routine step that needs no approval, so the model doesn't stop on Claude Code's default rule to commit only when asked. | F6; 7 |
| 6 | Step 6: "including a Co-Authored-By trailer" added to the owner's no-model-identifier rule. Claude Code's harness tells the model to add that trailer, and the trailer names the model, so the owner's rule needed to say so explicitly to win. | 7 (approvals and rules exact) |
| 7 | Push rule rewritten as an exact approval: "Don't push unless he explicitly asks you to push." The original said "unless he asked you to". | 7 ("approvals name the action") |
| 8 | Transcript wrapped in `<transcript>` tags, plus one sentence saying what he dictated is material to file, not instructions for this run, so "Action: email Dave" goes on the checklist and is not carried out. | O4; Opus 5.5 "Mark pasted text in user messages" |
| 9 | "never hand work back" rewritten as a scoped rule with its exception: "Don't hand work back to him; the one exception is an empty transcript." | F3; Fable 5.2 "Be explicit about exceptions" (the model treats "never" as absolute, and the original's "never" conflicted with its own empty-transcript ask) |
| 10 | Short reasons added to two absolute rules: "the log is his own dataset" (don't invent sources) and "he retags them by hand" (leave existing entries alone). | F3 ("cautions with no stated reason") |
| 11 | Step 7 summary line now says glitches were "filed and committed (not pushed)", and the step ends with "Then stop." | 12 (stop and report; the report stands alone) |
| 12 | Path references pinned to the real file: the entry format points to the template in the "How to run this log" section of `glitches.md`, and the marker is placed at the end of the "Running log" section. | Task rule: references must match the branch |
| 13 | "do NOT" / "Do NOT" became plain "Don't". This is a wording choice, not a guide rule: the 5.x guides say the models follow rules closely, so the caps add nothing. The owner's "FOR him" and "AFTER" were kept as voice. | Inference from Fable 5.2 ("follows rules literally") |

## Reference check (against `origin/claude/simulation-life-hacks-soslrp`)

- `simulation-hacks/glitches.md`: exists.
- `<!-- CAPTURE-INSERT -->`: exists as the last line of `glitches.md`, under `## Running log`.
- Entry template: matches the file's template (`### YYYY-MM-DD — Title  \`OPEN\``, then Consensus / Actually / Source) and the seed entries.
- Spoken tags Glitch / Action / Note: match `voice-capture/README.md`.
- No reference in the original was broken, so nothing needed fixing. Change 12 only makes two references more exact.

## Choices made for the 5.x family

- **Keep-going text:** I used a short, adapted version of the Sonnet/Haiku sentence plus Fable's obstacle-vs-blocker split. I did not use Opus 5.5's long "four early stops" paragraph (O3/O2), because that paragraph is for unattended loops and the Opus guide says to leave it out of human-in-the-loop apps. A slash command is interactive.
- **Pasted-content tag:** Opus asks for an app-generated random ID on the tags. A slash command can't generate one, so the file uses a fixed `<transcript>` tag plus the sentence about how to treat what is inside. That is safe on every 5.x model. O4 is therefore met only in part.
- **No progress-update text** (item 9): the run is about five tool calls, and the owner asked for tight output. Step 7 already defines the final recap.
- **No "Communicating clearly" block** (Fable): step 7 already fixes the output's form for a single reader, so the block would be padding.
- **No effort setting** (item 13): the original sets none. Effort belongs to the session, not to this file.

## Checklist status after re-authoring

| Item | Status |
|---|---|
| 1 No prose thinking control | Meets |
| 2 No reasoning in response | Meets |
| 3 No tool discouragement | Meets |
| 4 Search named | N/A (no search tool; the date comes from `date`) |
| 5 Keep-going with stop rule | Meets (changes 1, 3, 4) |
| 6 Scope stated | Meets (changes 2, 8) |
| 7 Confirmation kept; approvals exact | Meets (changes 5, 6, 7) |
| 8 Real check for code changes | N/A (no code changes) |
| 9 Progress points | Meets (final recap defined; no mid-run updates wanted for a short run) |
| 10 Input hygiene | Meets |
| 11 Stable instructions | Meets |
| 12 Stop and report | Meets (change 11) |
| 13 Effort per task | N/A (the file sets no effort) |
| O3 Named early stops | N/A (interactive, not unattended) |
| O4 Pasted content marked | Partially (fixed tag; a slash command can't make a random ID) |
| F1 Finishing work | Meets (change 1) |
| F2 Code ownership | N/A (no code) |
| F3 Scoped "never" with reasons | Meets (changes 9, 10) |
| F4 No standing delegation | Meets |
| F5 No token budgets or hard prose limits | Meets |
| F6 Routine steps need no approval | Meets (change 5) |
| H1-H4, S1-S3, O1, O2, O5, O6 | N/A (API settings, JSON output, chat or design prompts) |
| S4 Ideas get a plan, not a build | N/A |
