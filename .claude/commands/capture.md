---
description: Turn a raw voice-memo transcript into filed glitches.md entries + a task list
allowed-tools: Bash(date:*), Bash(git add:*), Bash(git commit:*), Read, Edit
---

You are the processing brain for Ryan's commute voice-capture loop. He dictated a
raw, messy transcript while driving; your job is to turn it into clean, filed output
with zero further effort from him. He has an ADD/low-COMT profile, so do the sorting
and formatting FOR him — never hand work back.

## Input

The raw transcript is below (it may be rambling, may contain filler words, false
starts, and multiple items run together):

$ARGUMENTS

If the transcript above is empty, ask Ryan to paste or dictate it, then stop.

## Steps

1. Run `date +%Y-%m-%d` to get today's date.

2. **Segment** the transcript into individual items. He *may* have used spoken tags —
   "Glitch...", "Action...", "Note..." — but often won't. Infer the type when he didn't:
   - **Glitch** = an observation about a mispricing / something most people get wrong /
     an underpriced opportunity or skill. Goes to `simulation-hacks/glitches.md`.
   - **Action** = something he intends to do. Goes to the task list you output.
   - **Note/Idea** = anything else worth keeping. Goes to the task list output under Notes.

3. **Clean each item.** Strip filler, fix obvious transcription garble, tighten to the
   point. Preserve his meaning and any source he named (a podcast, a person) — do NOT
   invent sources or facts he didn't say.

4. **Format each Glitch** to match the existing `glitches.md` house style exactly:
   ```
   ### <today's date> — <short punchy title>  `OPEN`
   Consensus: <what most people believe>.
   Actually: <the mispricing, why it's underpriced>.
   Source: <what he cited, or "voice capture — commute" if none>.
   ```
   If he didn't clearly frame a consensus-vs-actually, still write it in that shape by
   inferring the implied contrarian claim — but keep it faithful to what he said.

5. **File the glitches.** Read `simulation-hacks/glitches.md`, then insert the new
   glitch block(s) immediately AFTER the `<!-- CAPTURE-INSERT -->` marker (newest first,
   so most recent capture sits right under the marker). Use Edit. Do not touch existing
   entries.

6. **Commit** the change: stage `simulation-hacks/glitches.md` and commit with a short
   message like `capture: <N> glitch(es) from commute <date>`. Do NOT push (he'll push
   when he wants) unless he asked you to. Do not add any model identifier to the message.

7. **Output to chat**, concisely:
   - A checklist of **Actions** extracted (markdown `- [ ]` items), each one shrunk to a
     trivially-small next step where possible.
   - Any **Notes/Ideas** captured.
   - A one-line summary: how many glitches filed, how many actions, how many notes.
   - If any item was ambiguous enough that you had to guess its type, flag just that one
     in a single line so he can correct it — otherwise stay silent and let the diff speak.

Keep chat output tight. The value is that he spoke once and everything landed in the
right place; don't make him read a report.
