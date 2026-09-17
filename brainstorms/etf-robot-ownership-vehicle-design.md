# Vehicle Design: A New SPV / REIT Form for Robot Ownership

Companion to [`etf-style-robot-ownership.md`](./etf-style-robot-ownership.md). That doc
asked *what* to own. This one asks *through what legal form* — which turns out to be the
binding constraint, not the finance.

Not legal or tax advice. Every rule below needs counsel to confirm; tax details in
particular move.

---

## 0. The claim

**You probably don't need new law. You need three existing forms welded together in a way
nobody has bothered to do yet:** a *series/cell entity* for cheap per-unit legal separation,
a *passive owner trust* for tort isolation, and a *condominium-style governance layer* for
operator accountability. The genuinely missing pieces are an **asset registry** and a
**continuity covenant**, and neither requires Congress.

---

## 1. Why the REIT wrapper doesn't fit — and the two doors that are actually open

A REIT (IRC §856 et seq.) requires roughly: ≥75% of assets in real estate assets, ≥75% of
gross income from rents from *real property*, distribution of ≥90% of taxable income. Real
property means land plus inherently permanent structures and their structural components.

**A mobile robot is personal property. Full stop.** A humanoid that walks between sites is
about as far from "inherently permanent" as an asset gets. So a "robot REIT" holding
humanoids is a non-starter on the plain text.

But there are two doors, and both are open today.

### Door 1 — The affixedness gradient

Automation is not one asset class. It's a spectrum of how bolted-down it is:

| | Example | Real property? | Wrapper |
|---|---|---|---|
| **Fixed** | AS/RS racking, gantries, sortation, conveyor, port cranes | Plausibly yes — inherently permanent structure / structural component | **REIT-able, arguably today** |
| **Semi-fixed** | Fixed-path AGVs on embedded guidance, charging infrastructure, docks | Grey. Charging infra and embedded track look like structural components; the vehicle doesn't | Split title: infra in the REIT, vehicles in a TRS or side SPV |
| **Mobile** | AMRs, humanoids, drones, ag rovers | No | Trust / series LLC / leasing co |

The REIT definition has been stretched by private letter ruling and regulation for four
decades: timber, cell towers, billboards, prisons, casinos, and — most instructively —
**data centers**, where the fight over which components count as structural was settled
largely through PLRs and the 2016 "real property" regulations. **The historical pattern is
that new asset classes enter the REIT wrapper through rulings, not legislation.** Nobody
has yet filed the PLR asking whether a bolted-down AS/RS installation is an inherently
permanent structure. Someone will, and they'll probably win on the fixed end of the
gradient.

### Door 2 — Incidental personal property (the 15% rule) + the TRS

Two provisions do real work here:

1. **Rents from personal property leased *with* real property** count as good REIT income
   if the personal-property rent is at or below ~15% of the total rent for that property
   (§856(d)(1)(C)). So an industrial REIT can lease **a fulfillment center that comes with
   the robots in it**, as long as the robot rent stays inside the band.
2. **A taxable REIT subsidiary** can be up to ~20% of a REIT's assets and can own and
   operate what the REIT itself cannot.

Stack them and you get a legal, buildable-today product: **the automation-included
industrial REIT.** Lease the box plus the fleet inside it. The tenant gets turnkey
throughput instead of a shell; the REIT gets a higher rent per square foot and a genuinely
differentiated product; the investor gets partial robot exposure in a liquid, retail,
1099-issuing wrapper.

That is the single most executable idea in either of these two documents. It needs no new
law, no new asset class, and no new investor education — and existing industrial REITs are
already the natural issuers.

---

## 2. Why SPV design for robots is genuinely different

Aviation is the template, but three things invert.

### 2.1 Legal cost per unit
Aircraft finance uses **one owner trust per aircraft**. At $100M an asset, $40k of legal is
a rounding error. At $50k per robot, per-unit SPVs cost more than the asset.

→ **The structural innovation needed is the cell company.** Delaware Series LLC, Delaware
Statutory Trust series, or a protected-cell company: each series is a legally separate
pool of assets and liabilities at near-zero marginal cost.

This is not speculative. **Masterworks, Rally and the fractional-asset platforms already
run Series LLC + Reg A+ Tier 2** — one series per painting, per car, per collectible, each
with its own offering circular, sold to retail, no K-1. **That is the closest working legal
template for retail robot ownership that exists**, and the assets it was built for are
*worse* than robots in every respect (no cash flow, no telemetry, no contracted revenue).
The ceiling is the Reg A+ annual raise cap, which is why this is a launchpad, not a
destination.

### 2.2 The SPV's primary job is tort isolation, not creditor priority
In aviation, the owner trust exists mainly to structure creditor rights and registration.
For robots it exists mainly so that **when a machine hurts someone, the claim stops at the
operator and its insurer and never reaches passive owners.**

→ Design: bare legal title in a passive owner trust; all operational control contractually
vested in a licensed operator; trust named as additional insured; operator indemnity;
per-cohort cell so one incident can't cascade across the fleet. Passivity is not a
formality — an owner who directs operations forfeits the shield.

→ Corollary: **if the insurance market won't write the fleet, no SPV design saves you.**
Insurer appetite is the real gating factor, and it's the leading indicator to watch.

### 2.3 Repossession rights don't exist yet — except where they already do
An aircraft creditor has the **Cape Town Convention**: an international registry of
interests, priority rules, and IDERA-based repossession. A robot creditor has UCC Article 9
filings and a hope.

But note: Cape Town works by protocol per asset class — aircraft, rail rolling stock, space
assets, and the **MAC Protocol** covering **mining, agricultural and construction
equipment**, adopted at Pretoria in 2019. *(Check its current ratification / entry-into-force
status — it has moved slowly.)*

**Autonomous ag, mining and construction machines may therefore already sit inside a
purpose-built international asset-finance regime.** That is a strong, independent
confirmation of the other doc's conclusion that this asset class forms first in ag, mining
and heavy industry — not because the robots are better, but **because the law is already
built there.** For everything else, the concrete institutional ask is a **fifth protocol,
or a national serial-number registry, for high-value autonomous mobile equipment.**

---

## 3. The proposed form: an Automated Productive Asset Trust

Not really new — assembled from parts that all exist. Features, in rough order of how much
each one matters:

1. **Cell / series structure**, one cell per *cohort*: SKU × vintage × operator × contract
   tenor. Cheap separation, clean attribution, no cross-contamination.
2. **Defined maturity.** Each cell amortizes over the cohort's economic life, returns
   capital, and dissolves. No perpetual NAV to argue about — the exit is the design, not an
   afterthought. Robots depreciate like servers; the wrapper should behave like a
   target-maturity bond fund, not a REIT.
3. **Passive owner trust** holding title; operations vested elsewhere (§2.2).
4. **Continuity covenant** — the structural answer to brickability. Firmware and fleet-OS
   escrow, a perpetual irrevocable license triggering on operator default or insolvency, a
   pre-contracted standby operator, guaranteed data portability. Without this the trust owns
   scrap on the operator's first bad quarter, and bankruptcy remoteness is fictional.
5. **Attestation covenant** — units report hours, location, uptime and energy draw **to the
   trustee**, not only to the operator. Continuous proof of existence and proof of work.
   Defeats ghost-asset and double-pledge fraud structurally rather than by audit sampling.
6. **Independent valuation agent** using a third-party used-unit index. The sponsor never
   marks its own book.
7. **Sponsor risk retention**, ABS-style, so the operator can't syndicate its worst-utilized
   units.
8. **Insurance as first-loss**, trust as named insured, parametric downtime cover optional.
9. **Registered security interest** — MAC/Cape Town where eligible, serial-number UCC-1
   filings elsewhere.
10. **Governance rights**: for-cause operator termination, standby operator, reserve account,
    owner voting on reinvestment vs. wind-down.

---

## 4. The condominium analogy — a better mental model than REIT

Stop thinking REIT. **A robot fleet is a condominium.**

Condominium law already solves, by statute, exactly the problem here: individually-titled
units, shared common elements, a mandatory management association, assessments for upkeep,
reserve requirements, and — crucially — **a defined process for owners to replace the
management company.**

Map it:

| Condo | Fleet |
|---|---|
| Unit | Robot (or cohort cell) |
| Common elements | Charging infra, depot, fleet OS, spare parts pool |
| Association | Owner trust / fleet association |
| Management company | The operator |
| Assessments | Maintenance, energy, software subscription |
| Reserves | Refurbishment and replacement sinking fund |
| Bylaws | Operating agreement, utilization standards, safety protocol |

This reframing earns its keep because it foregrounds **governance**, which every
robot-ownership pitch ignores and which is the failure mode that actually shows up.
Externally-managed REITs trade at persistent discounts for exactly this reason: conflicted
external managers whom owners cannot fire. A robot trust with an external operator is the
same trap.

**Operator portability is the governance twin of the brickability problem** — and note that
#4 above is a technical precondition for #10. You cannot fire an operator you cannot
replace. If the fleet OS is the operator's, owner governance rights are decorative.

Dark cousin worth staring at: **timeshare.** Fractional retail ownership of a depreciating
asset with mandatory recurring fees, sold hard, with no exit. Every structural feature of a
badly-built robot trust is a timeshare. The differences that matter — real cash flow, real
telemetry, real defined maturity — must be structural, not promotional.

---

## 5. The tax fork, which determines who the product is for

| Wrapper | Entity tax | Depreciation | Retail? | Notes |
|---|---|---|---|---|
| **REIT** | None if 90% distributed | Trapped at entity | Yes, 1099 | Only via §1 doors |
| **Partnership / Series LLC** | Pass-through | **Flows to investors** — MACRS 5–7yr, bonus | K-1 friction; Reg A+ series avoids it | Best after-tax for taxable investors |
| **RIC (the actual ETF/mutual fund)** | None | N/A | Yes | **Requires securities, not machines** |
| **C-corp leasing co** | Double | Used at entity | Fully liquid | AerCap/Air Lease model |
| **BDC / interval fund** | Pass-through-ish | Limited | Yes, quarterly liquidity | Realistic Tier-2 home |

Three things fall out:

**(a) The RIC line is the crux.** A '40 Act fund needs ~90% of gross income from securities.
A fund holding *notes secured by robots* qualifies; a fund holding *robots* does not.
**So the path into a real ETF wrapper runs through securitization** — which is why Tier 1
(fleet ABS) is not merely the easiest product but the structural prerequisite for Tier 0.
The ABS market is the on-ramp, not a detour.

**(b) The depreciation shield inverts the usual account advice.** Accelerated depreciation is
a large share of after-tax return. A taxable investor uses it; an IRA cannot, and picks up
UBTI if there's leverage. **This is a yield-flavored asset that belongs in a taxable
account** — the opposite of nearly every other income product. Pension and foreign capital
need blocker structures, which pushes institutional money toward the C-corp and ABS forms
and leaves the equity to taxable investors.

**(c) Depreciation-led returns are a known trap.** See §6.

---

## 6. Two precedents that should scare anyone building this

**1980s retail equipment-leasing limited partnerships.** Diversified pools of
income-producing equipment, sold to retail through brokers, marketed on yield and
depreciation. The Tax Reform Act of 1986 gutted passive-loss deductions and the category
collapsed, leaving illiquid partnerships, disputed residuals, and years of litigation.
**This exact product has been tried, at retail, and it ended badly.** Anything built now has
to name the differences out loud: real-time telemetry, defined maturity, independent
valuation, and returns underwritten on cash yield rather than tax timing. If the pitch deck
leads with depreciation, it is the 1985 product with better graphics.

**Non-traded REITs.** Sponsor-marked NAV, high load, gated redemptions. Every one of those
failure modes is available to a robot trust by default. The defenses are §3's items 2, 6
and 7, and they have to be in the documents, not the marketing.

---

## 7. What to build first, concretely

1. **The automation-included industrial REIT** (§1, Door 2). Buildable now, by an existing
   REIT, with existing law. Lowest risk, real differentiation.
2. **Series LLC + Reg A+ cohort vehicle** on the Masterworks rails — one series per fleet
   cohort, defined maturity, telemetry-reported utilization. Proves the retail structure at
   small scale. Capped by the Reg A+ raise limit, which is fine for a pilot.
3. **The PLR.** Someone should ask the IRS whether a bolted-down AS/RS installation is an
   inherently permanent structure. The answer reprices the entire fixed-automation end of
   the gradient, and the question costs far less than the option is worth.
4. **The registry.** A serial-number registry of interests in autonomous equipment, or
   advocacy for a Cape Town protocol beyond MAC. Boring infrastructure, disproportionate
   unlock — and a natural home for the used-unit price index the other doc argues is the
   best standalone business here.

---

## 8. Open questions

1. Has anyone actually filed a PLR on affixed automation as real property? If not, why not —
   is there a known adverse authority?
2. Current MAC Protocol ratification status, and whether an autonomous ag rover is covered
   equipment under it.
3. Do Delaware Series LLC liability walls hold up against **tort** claims — as opposed to
   contract creditors — in a state that hasn't adopted series statutes? This is load-bearing
   for §2.2 and is genuinely untested at scale.
4. Will insurers write a fleet whose owner is a passive trust, or do they insist on an
   owner with operational control? If the latter, §2.2 collapses and the C-corp leasing
   company wins by default.
5. Can a continuity covenant survive the OEM's own bankruptcy, or does the escrow just
   become an asset of the OEM's estate?

*Question 4 is the one that decides whether any of this is real.*
