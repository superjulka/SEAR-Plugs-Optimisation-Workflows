---
name: meta-ads-monthly-review
description: >
  Monthly Meta Ads optimisation workflow for searplugs.com. Use this skill whenever
  it's time for the monthly Meta Ads review, when CPA is rising, when frequency is
  climbing, when creative feels stale, or when the user says anything like "check how
  the ads are doing", "time for the monthly review", "Meta ads need looking at",
  "optimise the campaigns", or "what should I do with the ads this month". Also use
  when an automated monthly trigger fires. The skill sequences all analysis steps and
  Adspirer tool calls in the right order to stay within the 15-call/month budget.
---

# Meta Ads Monthly Review — searplugs.com

This skill runs the monthly optimisation cycle for SEAR's Meta campaigns. It sequences
analysis work carefully: skills that use exported data (no API calls) come first, then
a small focused set of Adspirer calls, then a prioritised action list.

The 15 Adspirer call budget renews monthly. Treat it like a finite resource. This review
consumes 3–5 calls. The rest of the month's allowance is reserved for executing approved
changes and handling surprises.

---

## Before You Start

Ask the user two quick questions if the context doesn't already answer them:

1. **"Have you exported performance data from Meta Ads Manager this month?"**
   If yes — great, use it in Steps 1–3.
   If no — tell them: *"For the deepest analysis, export a campaign/ad-level report from
   Meta Ads Manager (last 30 days, include: impressions, clicks, CTR, spend, frequency,
   CPM, reach, purchases, cost per purchase). But we can also run the Adspirer checks
   without it — your call."*

2. **"Any specific concerns going in — CPA spike, a creative that's not working, a new
   ad set you want to add?"**
   Note these and prioritise them in the output.

---

## Step 1 — Skill-Based Analysis (no Adspirer calls)

Run these in sequence using exported Ads Manager data. Each produces findings that feed
into the final action list.

### 1a. Creative Fatigue Check → `/creative-fatigue-detection`

Pass ad-level performance data (last 14–30 days). Look for:
- Ads with frequency > 3.5 and declining CTR (urgent: replace now)
- Ads with frequency 2.5–3.5 and CTR declining 5–10%/day (warning: 1–2 weeks left)
- Ads that have been running > 5 weeks regardless of frequency (high exhaustion risk)

**Output:** List each ad as URGENT / WARNING / HEALTHY with supporting data.

### 1b. Audience Overlap Check → `/audience-overlap-analysis`

Run only if there are 2+ active prospecting ad sets. Pass ad set targeting parameters.
Look for:
- Overlap > 30% between any two ad sets (consolidate or add exclusions)
- CPM inflation signals correlated with overlap

**Skip this step** if running a single ad set — not applicable.

### 1c. Frequency Cap Review → `/frequency-cap-recommendations`

Pass campaign-level frequency and conversion data. Identify:
- The frequency inflection point where CVR starts declining
- Whether current frequency is above or below that point
- Recommended cap adjustments

---

## Step 2 — Adspirer Performance Pull (1–2 calls)

Now use Adspirer to pull live platform data. This is where you confirm or challenge what
the skill-based analysis found.

### Call 1: `get_meta_campaign_performance`

```
lookback_days: 30
```

Pull: spend, impressions, clicks, CTR, purchases, CPA, ROAS, CPM.
Present a clean table ordered by spend. Flag:
- Any campaign with CPA above your target (set your own — ours is internal)
- Any campaign with ROAS below your target
- CPM trends (rising CPM = audience saturation or creative fatigue signal)
- Budget pacing (is the full daily budget being spent? Under-delivery = audience too narrow
  or ad quality score too low)

### Call 2 (if CPA is above target or CTR is declining): `detect_meta_creative_fatigue`

Only use this call if Call 1 reveals a performance problem. If everything looks healthy,
skip it and bank the call.

Look for: frequency by ad, CTR trajectory, ad relevance scores. Cross-reference with
the `/creative-fatigue-detection` findings from Step 1a.

---

## Step 3 — Wasted Spend Check (1 call, conditional)

### Call 3 (if spend crossed your minimum-data threshold in the period): `analyze_meta_wasted_spend`

Use only if monthly spend has crossed your threshold — below that threshold there isn't enough
data for meaningful waste analysis. If spend is lower, skip and bank the call.

Look for: underperforming ad sets, placement-level waste, audience segments with high
spend and zero conversions.

---

## Step 4 — Synthesise Findings

Combine outputs from Steps 1–3 into a unified picture. Think about what's actually
driving performance rather than listing every data point. Group findings into:

**Creative health** — which ads need rotating, which are still strong, how many weeks
of useful life remain on current creative.

**Audience health** — is the targeting efficient? Any overlap or saturation signals?
Is the retargeting pool growing (more site visitors = more retargeting potential)?

**Budget efficiency** — is spend going to the right places? Any obvious waste?

**Tracking health** — any signs the pixel or CAPI data has degraded? Unexplained
conversion drops without corresponding traffic drops are a tracking signal.

---

## Step 5 — Action List (the output)

Produce a prioritised action list. Keep it short and specific — the user should be able
to action each item without ambiguity. Use this format:

```
## SEAR Meta Ads — Monthly Review [Month Year]
**Adspirer calls used this review:** [N] / 15 this month

### 🔴 Do This Week
- [Specific action] — [1-line reason]
- [Specific action] — [1-line reason]

### 🟡 Do This Month
- [Specific action] — [1-line reason]
- [Specific action] — [1-line reason]

### 🟢 Monitoring (no action needed)
- [Item] — [what to watch for]

### 💡 Next Month
- [Upcoming opportunity or test to plan]

### 📊 Performance Snapshot
| Metric | This Month | Target | Status |
|--------|-----------|--------|--------|
| CPA | £X | < [your target] | 🔴/🟡/🟢 |
| ROAS | X.Xx | > [your target] | 🔴/🟡/🟢 |
| Avg Frequency | X.X | < 4.0 | 🔴/🟡/🟢 |
| CPM trend | +/-X% | Stable | 🔴/🟡/🟢 |
| Creative health | X urgent / X warning / X healthy | — | — |
```

Keep each action item actionable in under 5 minutes of reading — if it requires more
context, link to the relevant skill (e.g., "Run /ad-copy-variant-generator on Angle 1
to generate 4 fresh variants").

---

## Adspirer Call Budget Tracker

Track calls carefully across the month. The 15-call budget renews on the 1st.

| Call | Tool | When to use |
|------|------|-------------|
| Monthly review | `get_meta_campaign_performance` | Always — 1st of month |
| Monthly review | `detect_meta_creative_fatigue` | If CPA rising or CTR declining |
| Monthly review | `analyze_meta_wasted_spend` | If spend crosses your threshold |
| Creative refresh | `update_meta_ad` or `add_meta_ad` | When URGENT fatigue confirmed |
| New ad set | `add_meta_ad_set` or `create_meta_image_campaign` | When expanding audiences |
| EU expansion | `search_meta_targeting` | When adding EU ad sets (Month 2+) |
| Retargeting | `update_meta_ad_set` | Adjusting retargeting window or budget |
| Lookalike | `add_meta_ad_set` | When testing lookalike audiences |

**Budget rule:** Never use more than 5 Adspirer calls in the monthly review itself.
Leave at least 10 calls for the rest of the month to action findings and test new things.

If you've already used calls this month for other tasks, check the remaining budget
before running this review and adjust which calls to make accordingly.

---

## Decision Logic for Common Situations

**"CPA is too high (above target)"**
1. Check creative fatigue first — declining CTR on high-frequency ads is the most common
   cause. If yes: rotate creative.
2. Check audience overlap — if multiple ad sets are competing, consolidate.
3. Check CPM — if CPM rose > 30%, it's a seasonal or audience saturation issue, not
   creative. Try widening the audience.
4. If all of the above look fine: review landing page (conversion rate on searplugs.com
   shop). The ad might be fine but the page is leaking conversions.

**"Frequency is above 4 and I'm worried"**
Run /frequency-cap-recommendations with the frequency-by-campaign data. If the analysis
confirms diminishing returns above your current frequency, set a cap in Meta Ads Manager.
This doesn't need an Adspirer call — frequency caps are set in the Meta UI.

**"An ad set isn't spending its full budget"**
This almost always means the audience is too narrow (Meta can't find enough people) or
the ad quality score is low (Meta is limiting delivery). Try broadening targeting or
refreshing the creative before increasing the budget.

**"Should I add an EU ad set this month?"**
Yes, if: the UK campaigns have been running 6+ weeks and are hitting CPA target, pixel
has 50+ purchase events, and you have creative that mentions EU pricing (€45.95). Use
1 Adspirer call (`search_meta_targeting`) to find EU interest targeting, then
`add_meta_ad_set` to create it.

**"When should I test lookalike audiences?"**
When pixel has 100+ purchase events. Before that point there isn't enough seed data for
Meta to build a quality lookalike. At a small daily budget this will take 2–4 months depending on
conversion rate.

---

## Quarterly Add-On (run every 3 months)

Once per quarter, run `/competitor-creative-analysis` alongside this review. No Adspirer
calls needed — it uses web research to scrape Meta Ad Library for competitor brands
(search: "surfer's ear earplugs", "waterproof earplugs", "swim earplugs", "surf earplugs").

Look for: new messaging angles competitors are testing, formats they're pushing (video
vs static), creative rotation frequency, gaps they're not filling.

Feed findings into the next creative brief as new angles to test or differentiate against.
