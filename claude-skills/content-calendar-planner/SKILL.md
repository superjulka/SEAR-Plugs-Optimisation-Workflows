---
name: content-calendar-planner
description: >
  Use this skill whenever the user wants to plan out a month or quarter of blog content for searplugs.com. Triggers include: "content calendar", "plan my posts", "what should I publish this month", "plan out content", "schedule blog posts", "create a content plan", "map out topics for the next month/quarter", or any request to organise upcoming blog content in advance. Also triggers when running the automated monthly planning pipeline. Always use this skill for content planning — do not freestyle a calendar without it.
---

# Content Calendar Planner — searplugs.com

This skill plans a structured blog publishing schedule for searplugs.com, maps each slot to a researched keyword, and outputs a calendar file that `seo-blog-writer` can read directly — either manually or as part of an automated pipeline.

This skill is part of the integrated searplugs.com SEO system defined in the **SEAR Plugs Automated SEO Strategy** doc. It sits downstream of `keyword-researcher` (which produces the prioritised keyword pool) and upstream of `seo-blog-writer` (which turns each calendar row into a WordPress draft). The shared handoff file is the single source of truth for the pipeline.

---

## Non-negotiables (read before every run)

These rules override anything else in this skill.

1. **No "custom" language anywhere in the calendar.** Never plan a post whose primary keyword, suggested title, angle, or content notes contains "custom earplugs", "custom-fit", "custom-moulded", "bespoke", or "made-to-fit". These terms misrepresent SEAR. Correct product-aligned anchors: "waterproof earplugs", "surf earplugs", "swim earplugs", "water sports earplugs", "earplugs for surfer's ear", "earplugs for swimmer's ear". See the `product_sear_earplugs` auto-memory for the full product fact sheet.
2. **Medical-adjacency governance.** Mark any slot whose topic covers medical symptoms, diagnoses, or treatment (surfer's ear surgery, otitis externa treatment, hearing-loss prognosis, etc.) with `Medical-adjacent: Yes` in the calendar row. `seo-blog-writer` uses this flag to enforce draft-only governance and avoid unsupported clinical claims. Do **not** plan surgical-/treatment-intent primary keywords (e.g. "exostosis surgery cost") — wrong audience, wrong commercial fit.
3. **No competitor brand keywords.** Never plan a post targeting a competitor brand name (your category's named alternatives).
4. **Cap medical-adjacent slots at 1 per month.** Over-indexing on clinical content creates governance overhead and shifts the brand away from commercial positioning. One slot max.
5. **Reserve at least 1 EU-targeted content slot per month.** SEAR ships to Europe and EU organic growth is a top strategic priority. Every monthly calendar must include at minimum one post that is explicitly EU-targeted — either written for a European-language audience (DE, ES, FR, PT, NL) or centred on a European surf/swim location and audience. Mark these slots with `EU Target: Yes` and note the target market. If keyword-researcher has produced Sheet 5 (EU Language Keywords), draw from it; otherwise plan English-language posts with EU surf/swim angle (e.g. Hossegor, Peniche, Mundaka, Scheveningen, Zarautz).

---

## What this skill produces

- A publish calendar for the requested period (default: current month + next month)
- Each slot mapped to: publish date, primary keyword, secondary keywords, content type, audience angle, suggested title, EU target flag, and medical-adjacency flag
- Seasonal relevance and content balance factored in (informational vs commercial, surfer vs swimmer audience, UK vs EU market)
- An `.xlsx` calendar file saved to a fixed path the `seo-blog-writer` can find automatically
- A brief verbal summary of the plan rationale

---

## Inputs

**Required:**
- Time period (e.g. "May 2026", "Q3 2026", "next 6 weeks")

**Optional:**
- Post frequency (default: **2 posts per week**, Mon + Thu)
- Seasonal theme or campaign focus (e.g. "summer swim season", "surf season", "EU expansion push")
- Specific keywords to include (overrides keyword research for those slots)
- Whether this is a manual run or automated pipeline run (affects output verbosity)

---

## Step-by-step workflow

### Step 1 — Gather keyword inputs

Good calendar planning starts with keywords, not topic ideas. Check these sources in order:

**1a — Existing keyword-researcher output**

Look for keyword research spreadsheets in the outputs folder, named like `keyword-research-[topic]-[date].xlsx`. If found, read:
- Sheet 1 (Keyword Research) — extract High-priority keywords where `Existing on SEAR?` is No or Partial; note EU Market Language column for EU-targeted slots
- Sheet 4 (Filtered Out) — scan it so you never plan a keyword that was explicitly excluded
- **Sheet 5 (EU Language Keywords)** — pull the top EU-language keyword opportunities per market to fill the mandatory EU slot(s). Prioritise markets with the most strategic fit based on current GSC signals (DE, ES, FR, PT, NL in rough order).

If multiple keyword-research files exist for different seeds, combine their High-priority rows into a single candidate pool.

**1b — WordPress content inventory**

Use `wp_posts_search` to pull all published posts. Note their titles and primary-keyword mappings so the calendar doesn't duplicate existing content. Partial-coverage topics are good candidates for new posts.

Cross-check against `mcp__gsc__cannibalization` — if two SEAR URLs already compete for a keyword, do not plan a third; flag for consolidation instead.

**1c — GSC quick-win topup**

Before deciding the final mix, pull `mcp__gsc__quick_wins` and `mcp__gsc__opportunity_finder`. Any keyword appearing there with GSC position 8–20 and meaningful impressions that isn't already on the candidate list should be added — these are the highest-confidence slots to win new traffic. Tag the source as `GSC quick-win` in the calendar's Content Notes column.

**1d — Fill gaps with web search**

If the pool is still sparse, run a quick web search on core SEAR topics (surfer's ear prevention, swimmer's ear prevention, earplugs for surfing, earplugs for open water swimming) to surface high-value keywords not yet planned. **Do not include** "custom earplugs" variants — drop and note (Non-negotiable #1).

---

### Step 2 — Factor in seasonality and content balance

Before assigning keywords to dates, apply these filters.

**Seasonal relevance — UK & Europe combined:**

- **Jan–Mar**: cold-water surfing season (UK Atlantic coast, Basque Country, Galicia, Portuguese coast). Good for: surfer's ear prevention, cold-water ear protection, recovery/year-round wear content. EU angle: Mundaka winter swells, Porto surf season.
- **Apr–Jun**: surf season ramps up across UK and Europe; open-water swim season begins. Good for: prevention guides, "getting ready for the season" content, swim earplugs, triathlon training prep. EU angle: Hossegor spring, Scheveningen season opens, Peniche swells.
- **Jul–Sep**: peak swim and surf season UK + Europe. Good for: product-comparison content (without naming competitor brands), parent/child swimming, triathlon season, open-water events. EU angle: Hossegor pro events, Sardinia diving/snorkelling, Mediterranean swim season, Lacanau Ocean Race.
- **Oct–Dec**: end of season, gifting period, new surfer onboarding. Good for: gift guides, beginner guides, year-round protection content. EU angle: Peniche WSL stop (Oct/Nov), Nazaré big-wave season, Portuguese autumn surf.

**Content balance (across the calendar period):**
- Aim for roughly 60% informational, 40% commercial intent
- Rotate audience across the calendar — don't cluster at the same audience back to back. Rotate through: surfers, swimmers, snorkelers, freedivers, triathletes, water polo players, paddleboarders, parents of young swimmers, ENT-referred patients, kitesurfers, wingfoilers, sailors, EU market (each country as a distinct audience)
- Space out closely related topics — never two ear-health-prevention posts in the same week
- Include at least one FAQ-rich post per month (these tend to pick up People Also Ask snippets)
- Include lifestyle and destination content (e.g. "Best UK surf spots for cold-water winter sessions", "Surfing Hossegor: What Every Wave Rider Needs to Know", "Top open-water swim spots in Europe") — these attract a wider audience and earn natural backlinks
- **Cap medical-adjacent slots at 1 per month** (Non-negotiable #4). Flag with `Medical-adjacent: Yes` and set Content Notes to remind `seo-blog-writer` about the draft-only marker.
- **Reserve at least 1 EU-targeted slot per month** (Non-negotiable #5). Mark with `EU Target: Yes` and specify the target market (DE / ES / FR / PT / NL / EU-general). Draw from Sheet 5 of the keyword-researcher spreadsheet if available.
- If a pillar topic (e.g. "surfer's ear prevention") has no dedicated post yet, prioritise it over long-tail variants

**Title discipline — avoid these patterns:**
- No "Complete Guide" or "Complete Buying Guide" in titles — overused and generic
- No "Ultimate Guide to..." — same problem
- No two posts with the same structural formula in the same month (e.g. "Best X for Y: [subtitle]" twice)
- No "custom" / "custom-fit" / "bespoke" in any title
- Every title should feel like a magazine headline — specific, intriguing, and useful
- Check existing WordPress post titles before finalising — never duplicate a title pattern already published

---

### Step 3 — Assign keywords to publish slots

Generate the publish schedule:
- Default cadence: **Monday and Thursday** of each week
- Start from the first Mon/Thu on or after the period start date
- Fill each slot with the best-fit keyword given seasonality, balance, and medical-adjacency caps
- Ensure the mandatory EU slot is placed — typically mid-month so it doesn't get cut if the calendar is shortened
- For each slot, also assign 2–3 secondary keywords from the research

Build this as a structured list before creating the spreadsheet:

```
[Date] | [Primary keyword] | [Secondary keywords] | [Intent] | [Audience] | [Suggested title] | [EU Target Y/N] | [EU Market if Y] | [Medical-adjacent Y/N] | [Notes]
```

---

### Step 4 — Build the calendar spreadsheet

Use the xlsx skill to produce the calendar file. Save it to a **fixed, predictable path** so `seo-blog-writer` and the scheduled tasks can find it automatically:

**Fixed save path:** `/sessions/<current-session>/mnt/outputs/content-calendar-active.xlsx`

Also write a timestamped copy alongside it for archival: `content-calendar-[YYYY-MM-DD].xlsx`. Overwrite `content-calendar-active.xlsx` each time a new calendar is planned — it's the single source of truth for the pipeline.

**Sheet 1: Publishing Schedule** (the main working sheet)

| Column | Description |
|--------|-------------|
| Publish Date | YYYY-MM-DD format |
| Day | Mon / Thu |
| Primary Keyword | Exact target keyword |
| Secondary Keywords | Comma-separated, 2–3 |
| Intent | Informational / Commercial / Transactional |
| Audience | Surfers / Swimmers / Parents / Triathletes / EU-DE / EU-ES / EU-FR / EU-PT / EU-NL / etc. |
| Suggested Title | Draft H1 — can be refined when writing |
| EU Target? | Yes / No |
| EU Market | DE / ES / FR / PT / NL / EU-general (blank if EU Target = No) |
| Medical-adjacent? | Yes / No |
| Content Notes | Angle, what to emphasise, seasonal hook, GSC signal if any, EU surf location if relevant |
| Status | `Planned` (default) / `In Progress` / `Draft in WP` / `Published` |
| WP Post ID | Blank until seo-blog-writer fills it in |
| WP Draft URL | Blank until seo-blog-writer fills it in |
| Last Updated | ISO timestamp — updated whenever Status changes |

Sort by Publish Date ascending.

**Sheet 2: Keyword Pool**
All keywords considered but not yet scheduled — useful for future months. Columns: Keyword | Intent | Priority | EU Market Language | GSC Impressions | GSC Position | Source | Notes

**Sheet 3: Archive**
Rows moved here once Status = Published. Keeps Sheet 1 clean.

**Sheet 4: Excluded**
Keywords filtered out (brand non-compliance / competitor brand / medical over-claim). Carried forward from `keyword-researcher`'s "Filtered Out" sheet so Julia has a running record.

**Sheet 5: EU Slots Summary** ← new mandatory sheet
One row per EU-targeted slot planned. Columns: Publish Date | Target Market | Primary Keyword (native or EN) | Language | Suggested Title | Notes

---

### Step 5 — Log the run and report back

Log the planning run to `ops_seo.actions` (if BigQuery is available):

```sql
INSERT INTO `YOUR_GCP_PROJECT.ops_seo.actions`
  (action_ts, action_type, url, detail)
VALUES
  (CURRENT_TIMESTAMP(), 'content_calendar_planned', NULL,
   JSON '{"period":"...","slots":N,"medical_adjacent_slots":N,"eu_targeted_slots":N,"informational_pct":N,"commercial_pct":N}');
```

If `ops_seo.actions` doesn't exist yet or BigQuery is unreachable, skip silently and note the skip in the report.

Then give a concise verbal summary:

```
Content calendar planned: [period]

[N] posts scheduled ([start date] → [end date])
Cadence: Mon + Thu

The plan:
[Date] — "[Suggested Title]" ([intent], [audience])[, 🌍 EU: [market] if applicable][, ⚕️ Medical-adjacent if applicable]
[Date] — "[Suggested Title]" ([intent], [audience])
... (list all)

Seasonal focus: [1 sentence]
Content balance: [X]% informational, [Y]% commercial
EU-targeted slots: [N] (minimum 1/month ✓)
Medical-adjacent: [N] slot(s) (cap: 1/month)

Calendar saved: content-calendar-active.xlsx (+ archival copy)
Action logged to ops_seo.actions: [yes/no]

Next step:
- Review the plan, adjust any titles or dates, then either:
- Trigger `seo-blog-writer` manually for each post when ready, or
- Let the automated `write-blog-draft` scheduled task pick up each row on its Publish Date
```

---

## Automated pipeline integration

This skill is designed to work as part of a two-tier automated schedule:

### Tier 1 — Monthly planning (scheduled task: `monthly-content-planning`, runs 1st of each month)

Runs:
1. `keyword-researcher` on core SEAR topics to refresh the keyword pool (with brand-rule exclusions, GSC signal grounding, **and EU language keyword expansion**)
2. `content-calendar-planner` for the coming month, using the fresh keyword data — ensures at least 1 EU slot is reserved
3. Saves `content-calendar-active.xlsx` to the outputs folder
4. Logs to `ops_seo.actions` and emails Julia a summary

### Tier 2 — Post writing (scheduled task: `write-blog-draft`, runs Mon + Thu mornings)

Runs:
1. Reads `content-calendar-active.xlsx`, finds the next row where Status = `Planned` and Publish Date ≤ today
2. Invokes `seo-blog-writer` with that row's primary/secondary keywords, audience, suggested title, **and EU Target / EU Market flags**
3. `seo-blog-writer` enforces non-negotiables (brand rule, medical-adjacency marker, draft-only, cannibalization check) and — when EU Target = Yes — includes EU location references and EUR pricing in CTAs
4. Updates the calendar row: Status → `Draft in WP`, fills in WP Post ID, WP Draft URL, Last Updated timestamp
5. Logs the publish action to `ops_seo.actions` with `eu_targeted` flag
6. Julia reviews the draft in WordPress and publishes when ready

### Why this split works
- Julia reviews the monthly calendar before any writing starts — she can swap topics, adjust titles, or pause the pipeline by setting Status to anything other than `Planned`
- Posts are written one at a time, which is more reliable and easier to quality-check than a batch
- The shared xlsx file is the single source of truth — both skills read and write to it
- Non-negotiables are enforced at every step, so drift between conversations is impossible
- The EU slot in every calendar ensures consistent EU market coverage without requiring Julia to remember to ask for it

---

## Things to avoid

- Don't schedule two posts on the same keyword or very similar topics in the same month
- Don't fill the calendar with only informational content — commercial intent posts drive conversions
- Don't ignore what's already published — always check WordPress first to avoid duplication
- Don't use vague title placeholders like "Post about swimmer's ear" — every suggested title should be compelling enough to publish as-is
- Don't include "custom earplugs" or competitor brand keywords anywhere in the calendar
- Don't exceed 1 medical-adjacent slot per month
- Don't skip the EU slot — it is mandatory per Non-negotiable #5. If a month has no good EU-language keyword candidate, plan an English-language post centred on a European surf/swim destination instead.

---

## Example invocations

**Manual use:**
> "Plan my content calendar for May 2026"

> "Create a content plan for Q3 — summer swim season focus, 2 posts a week"

> "What should we publish over the next 6 weeks? Use the keyword research I ran last week."

> "Plan June's calendar with a strong EU push — I want French and German content this month"

**Automated pipeline (called by `monthly-content-planning` scheduled task):**
> "Run monthly content planning: keyword research on core SEAR topics + calendar for the coming month, 2 posts per week (Mon + Thu), save to content-calendar-active.xlsx in outputs. Include EU language keyword research and reserve at least 1 EU-targeted slot."
