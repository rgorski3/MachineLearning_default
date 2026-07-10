# Voice Capture: the commute → repo loop

*The keystone habit from your [operating system](../simulation-hacks/OPERATING-SYSTEM.md). Goal: convert the one solo, uninterrupted block your life reliably gives you — the drive — into your only reliable output channel, at near-zero executive-function cost. You speak once, hands on the wheel; everything lands in the right place later.*

## The whole loop in one line

**Speak → auto-transcribe → paste into `/capture` → it files everything and commits.**

Three of those four steps are done by a machine. Your only job is the first one.

---

## Step 1 — Capture (in the car, hands-free)

**Use your phone's built-in assistant, not a new app.** The single biggest failure mode here is adoption friction: a new subscription app you have to remember to open and tap is dead on arrival for an ADD brain in a moving car. The assistant is already there, it's genuinely hands-free, and it's the safe choice while driving.

- **iPhone:** *"Hey Siri, take a note"* → talk → it saves to Apple Notes. (Or *"Hey Siri, new voice memo"* — Voice Memos now auto-transcribes.)
- **Android:** *"Hey Google, take a note"* → talk. (Or open **Google Recorder**, which transcribes live and is excellent — one tap before you pull out, then fully voice.)

**Upgrade, only once the habit sticks:** a ramble-to-clean-note app like **AudioPen** or **VoicePal** — purpose-built to turn a spoken mess into a tidy summary. Don't start here. Earn the upgrade by proving you'll do the basic version for two weeks first.

### The spoken grammar (so your brain doesn't have to decide format at 70 mph)

Start each item with one word. That's the entire system:

- **"Glitch —"** then the mispricing. *"Glitch: everyone thinks X, but actually Y, from the Latent Space episode."*
- **"Action —"** then the thing to do. *"Action: email Dave about the harness idea."*
- **"Note —"** then anything else worth keeping.

If you forget the tag, it still works — `/capture` infers the type. The tag just makes it cleaner. **One breath per item. Don't compose. Don't edit. Ramble is fine** — cleanup is the machine's job, not yours.

---

## Step 2 — Process (later, at a keyboard — 30 seconds)

Whenever you're back at your machine (or in the Claude app), open this repo and run:

```
/capture <paste your transcript here>
```

The `/capture` command ([.claude/commands/capture.md](../.claude/commands/capture.md)) will:
1. Split your ramble into separate items and clean each one up.
2. Sort them: **glitches** get formatted and filed into `simulation-hacks/glitches.md`; **actions** and **notes** come back to you as a checklist.
3. Commit the glitches automatically.

You never hand-edit the file. You never format anything. You paste a mess; structured output comes out.

> **No transcript text handy?** Most voice apps let you share/copy the transcript. If you only have audio, drop the file in and ask Claude Code to transcribe it first, then `/capture`. The native-assistant route above gives you text directly, which is why it's the recommendation.

---

## Step 3 — Actions go where you already look

`/capture` returns your actions as a checklist. Send them wherever you actually check —
your Notion **My Tasks**, a reminders app, whatever. (Say the word and I'll wire `/capture`
to push actions straight into your Notion task database so even that step disappears.)

---

## Why this beats "just journal on your commute"

- **No blank page, no initiation cost** — the two things ADD hates most. You're already in the car with your hands busy; you're just talking.
- **Zero formatting burden** — the inverted-U says your prefrontal cortex is your scarce resource under load; this spends none of it. The machine does the tidying.
- **It compounds** — every drive adds to `glitches.md`, turning listening you already do into a proprietary dataset (Exploit 9). Review it monthly; any glitch you capture three times is your next project.

## The two-week test

Do only Step 1 every drive, and Step 2 whenever convenient. Don't optimize, don't add the fancy app, don't build anything else. If in two weeks the running log in `glitches.md` has entries in it, the loop works and we upgrade it. If it's empty, the friction is still too high and we simplify further — that's data too.
