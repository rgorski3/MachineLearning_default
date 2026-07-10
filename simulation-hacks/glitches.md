# glitches.md — a running log of mispricings

*Exploit 9 from [the field guide](README.md), operationalized. You already track the AI frontier daily; this file converts that listening into a proprietary dataset. One entry per mispricing you notice: something the market/most people haven't priced correctly yet — an undervalued skill, a boring niche, a workflow that works better than the consensus realizes, a gap between hype and reality.*

## How to run this log

- **One entry per glitch, dated, 1–3 lines.** State what most people believe vs. what you think is actually true. Cite where you saw it. Don't polish — volume beats craft here.
- **Add entries within 48 hours of noticing**, or they evaporate. A snip you saved is a candidate entry; the entry is what the snip *implies* that others haven't acted on.
- **Tag each entry**: `OPEN` (unexploited), `PLAYED` (you acted on it — link the action), `EXPIRED` (market caught up / turned out wrong — keep it, wrong calls are calibration data).
- **Review monthly.** Look for entries that keep recurring in different forms — a mispricing you've independently noticed three times is your next project.

Entry template:

```
### YYYY-MM-DD — Title  `OPEN`
Consensus: what most people believe.
Actually: the mispricing. Why it's underpriced.
Source: episode/article/conversation.
```

---

## Seed entries

*Extracted 2026-07-10 from your own saved snips (May–July 2026). You already flagged all of these by saving them — this just names the mispricing in each.*

### 2026-07-08 — The first AI labor shock is independence, not unemployment  `OPEN`
Consensus: AI-exposed jobs means mass layoffs.
Actually: solo self-employment in AI-exposed occupations is up ~20% since 2022 (flat elsewhere); million-dollar solopreneurs more than doubled 2023→2025; 63% of new Q2-2026 C-corp filings are single-founder — an all-time high. Workers are migrating to independence before the layoff narrative even lands.
Source: "AI Is Making One-Person Million-Dollar Companies More Common" — The AI Daily Brief.

### 2026-07-08 — Headcount has decoupled from valuation  `OPEN`
Consensus: you need a team to look "serious" to investors and customers.
Actually: Harvard/INSEAD found AI-native startups run ~25% smaller and flatter yet are valued the same as traditionally-staffed peers. Small is no longer discounted.
Source: same AI Daily Brief episode, citing an SSRN/Harvard-INSEAD study.

### 2026-06-25 — Domain expertise + taste beats model access  `OPEN`
Consensus: best model wins; young AI-native hackers eat the incumbents.
Actually: Webflow co-founder's new startup (Ploy) wins on a curated 3,500-prompt design catalog and decades of taste steering a generic model — the same model produces slop for an unskilled user. Experienced domain experts are the ones compounding fastest.
Source: "Why Domain Experts Are Winning In The Age Of AI" — YC Lightcone.

### 2026-06-24 — Your workflow is portable IP nobody is capturing  `OPEN`
Consensus: the thing to own is AI "memory"/context.
Actually: your working procedure — how you research, verify, ship — is becoming executable software, and right now it dies in a Cursor-rules file on every tool switch. Portable skill/runbook files are wide open and mostly untried.
Source: "Your skills are leaving your hands. Don't let a rent-a-brain keep them." — Nate's Notebook.

### 2026-06-18 — Steer-vs-dispatch is a skill, not a tool war  `OPEN`
Consensus: Claude Code vs. Codex is a benchmark fight.
Actually: the real divergence is behavioral — steering agents closely vs. dispatching and auditing later — and "is this AI-delegated work real?" is becoming a discrete managerial skill nobody teaches yet. Whoever codifies it first owns a curriculum.
Source: "Claude vs. Codex isn't about code. It's about whether you steer or dispatch." — Nate's Notebook.

### 2026-07-09 — Agent experience (AX) is DX in 2010  `OPEN`
Consensus: cloud infra is mature and commoditized.
Actually: Modal's CTO is rebuilding infra around agent needs — fast iteration loops, observability over code-reading, sandboxes bursting to 100k for RL — and almost no infra team has made the switch. The category is being invented in public right now.
Source: "Why AI Infrastructure must evolve for Agent Experience — Akshat Bubna" — Latent Space.

### 2026-07-09 — Verification harnesses > code review  `OPEN`
Consensus: senior engineers add value by reading and critiquing code.
Actually: "people aren't looking at code" — leverage is shifting to observability, logs, and proof-of-done. Building verification harnesses is the unglamorous skill nobody is competing for.
Source: same Modal episode; reinforced by "proof of done" snip, Nate's Notebook 2026-06-24.

### 2026-06-11 — The non-engineer harness gap is ~10x wide  `OPEN`
Consensus: AI adoption is everywhere now.
Actually: Brex's CEO says engineering "token-maxers" get outsized productivity from coding harnesses while everyone else chats with a bot plus a few MCPs — roughly a tenth the output. Self-bootstrapping harnesses for non-engineers are barely attempted. This one is directly buildable from a day job.
Source: "The CEO Must Be the Chief AI Officer" — YC Lightcone (Brex).

### 2026-06-11 — AI raises the value of minimal surface area  `OPEN`
Consensus: AI lets you build more, so build more.
Actually: Stripe (API-only), early Brex (terminal-only), Airbnb (a form) won by compressing to one interaction — "intelligence is compression." AI makes feature sprawl cheaper and therefore less differentiating; discipline is the scarce input.
Source: same Brex/YC episode.

### 2026-06-25 — The mushy middle of AI adoption is the worst seat  `OPEN`
Consensus: partial, hedged AI adoption is the prudent path.
Actually: Hormozi's barbell — go fully AI-native on one end, or bet on durable non-automatable categories (healthcare, food, housing, entertainment) on the other. The moderate middle where most orgs sit is the worst-positioned place, and most orgs sit there.
Source: "The Barbell Strategy for Surviving the AI Shift" — The Game.

### 2026-05-11 — The reviewer-only software company already works  `OPEN`
Consensus: maintaining shipped software requires a team.
Actually: Naval runs Claude on a 24-hour cron that triages and auto-fixes every bug report into review branches; he's only the merge gatekeeper. A concrete, replicable operating pattern almost no solo builder has adopted past v1.
Source: "A Return to Code" — Naval.

---

## Meta-glitch (free entry)

### 2026-07-10 — Everyone archives, nobody extracts  `PLAYED`
Consensus: saving highlights is knowledge management.
Actually: a highlight archive is inventory, not insight — the extraction step (naming the mispricing and dating it) is nearly free and almost nobody does it. This file is the play.
Source: your own Notion, examined.

---

## Running log

*New entries land here from voice captures (see [../voice-capture/README.md](../voice-capture/README.md) — or just run `/capture` and paste a transcript). Newest first.*

<!-- CAPTURE-INSERT -->

