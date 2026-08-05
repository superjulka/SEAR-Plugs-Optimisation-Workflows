---
name: keyword-researcher
description: >
  Use this skill whenever the user wants to discover new keyword opportunities for searplugs.com blog posts or product pages. Triggers include: "keyword research", "what should I write about", "find keywords", "keyword ideas", "what are people searching for", "content opportunities", "what topics should we target", or when the user gives a seed topic and wants to know how to rank for it. Also triggers when the user mentions a competitor URL and wants to understand what keywords they rank for. Always use this skill before starting keyword research — do not freestyle keyword lists without it.
---

# Keyword Researcher — searplugs.com

This skill discovers, organises, and prioritises keyword opportunities for searplugs.com. It maps search intent, uses live GSC + BigQuery signals to ground prioritisation in what the site actually ranks for, and outputs a structured spreadsheet plus blog-title suggestions — so every piece of content has a clear keyword strategy before a word is written.

This skill is part of the integrated searplugs.com SEO system defined in the **SEAR Plugs Automated SEO Strategy** doc. It is the canonical upstream step for both `content-calendar-planner` and `seo-blog-writer`, and the spreadsheet it produces is the handoff artifact those skills read.

---

## Non-negotiables (read before every run)

These rules override anything else in this skill.

1. **Exclude "custom earplugs" and variants from the keyword universe.** SEAR earplugs are NOT custom, bespoke, or custom-moulded — they are waterproof earplugs sized via a 3-tip + 4-wing system. Even though "custom earplugs" is a real search term with volume, ranking SEAR content for it would mislead users and attract the wrong audience. Actively **filter out**: "custom earplugs", "custom-fit earplugs", "custom-moulded earplugs", "bespoke earplugs", "made-to-fit earplugs", "mould-your-own earplugs", "audiologist earplugs". If a seed or competitor research surfaces these, drop them with a one-line note in the report. See the `product_sear_earplugs` auto-memory for the correct product descriptors.
2. **Correct primary keyword anchors** (from the product fact sheet): "waterproof earplugs for surfing", "waterproof earplugs for swimming", "earplugs for surfing", "swim earplugs", "surf earplugs", "earplugs for surfer's ear", "earplugs for swimmer's ear", "water sports earplugs". Start keyword expansion from this list when the user gives a product-aligned seed.
3. **Flag medical-adjacent keywords.** Keywords targeting symptoms, diagnosis, or treatment (e.g. "surfer's ear surgery recovery", "ear infection from swimming treatment") must be flagged with `medical_adjacent: true` in the output — `seo-blog-writer` uses this flag to enforce draft-only governance and avoid unsupported clinical claims.
4. **No competitor brand names as target keywords.** Drop keywords containing competitor brand names (your category's named alternatives) — note them in a separate "Competitor brand queries — do not target" list for awareness only.
5. **EU language keyword research is mandatory.** SEAR ships to Europe and EU organic growth is a top priority. For every seed topic, always generate the native-language equivalents for the top 5 EU markets. These go into Sheet 5 (EU Language Keywords) and the EU Market Language column in Sheet 1. Priority markets and their key terms:
   - 🇩🇪 **DE**: "Ohrstöpsel Surfen", "Ohrstöpsel Schwimmen", "Surfer Ohr", "Wasseraktivitäten Ohrstöpsel", "Schwimmer Ohr Schutz"
   - 🇪🇸 **ES**: "tapones de surf", "tapones para nadar", "oído del surfista", "protección auditiva surf", "tapones oídos deportes acuáticos"
   - 🇫🇷 **FR**: "bouchons d'oreille surf", "bouchons natation étanches", "oreille du surfeur", "protection oreilles natation", "bouchons sports nautiques"
   - 🇵🇹 **PT**: "tampões de surf", "tampões para nadar", "ouvido do surfista", "proteção auditiva surf", "tampões desportos aquáticos"
   - 🇳🇱 **NL**: "surf oordoppen", "zwem oordoppen", "surfer oor", "waterdichte oordoppen", "oorbescherming zwemmen"

   Note: searplugs.com currently has near-zero EU non-brand organic traffic. EU language keywords are strategic bets — document them in the spreadsheet even if GSC shows no impression data yet.

---

## What this skill produces

- A keyword list grouped by search intent (informational, commercial, transactional)
- Recommended content format for each keyword (blog post, product page, FAQ, landing page)
- Quantified signals from GSC (impressions, current position, CTR) for keywords where searplugs.com already has visibility, supplemented by directional signals (autosuggest, PAA, forums) for net-new keywords
- Medical-adjacency flag per keyword
- EU market language equivalents for every major topic cluster
- 3–5 suggested blog post titles for the highest-opportunity keywords
- An `.xlsx` spreadsheet deliverable with all findings, in a schema `content-calendar-planner` and `seo-blog-writer` can read directly

---

## Inputs

**Required:**
- Seed topic or product area (e.g. "swimmer's ear", "surf earplugs", "surfer's ear prevention")

**Optional:**
- Competitor URL to reverse-engineer (e.g. a rival earplug brand's blog)
- Focus area: "blog content only", "product pages only", or "both"
- Depth: "quick scan" (10–20 keywords) or "deep dive" (40–60 keywords)
- EU language focus: "German only", "French and Spanish", etc. — if omitted, cover all 5 priority markets

---

## Step-by-step workflow

### Step 1 — Pull GSC baseline (what SEAR already shows up for)

Before expanding anything new, capture what searplugs.com already has impressions/clicks for. This grounds prioritisation in real data rather than gut feel.

- `mcp__gsc__search_analytics` — pull queries for the last 90 days filtered to the seed topic (e.g. queries containing "surfer's ear", "swim earplugs"). Capture: query, impressions, clicks, CTR, average position.
- `mcp__gsc__opportunity_finder` — surface queries where SEAR is at position 8–20 with meaningful impressions. These are the best quick wins.
- `mcp__gsc__position_tracking` on the seed's top 5–10 queries — shows trajectory (rising/flat/declining)
- `mcp__gsc__cannibalization` filtered to the seed topic — flag queries where multiple SEAR URLs are competing
- **EU GSC check**: use `mcp__gsc__device_country_breakdown` or `mcp__gsc__search_analytics` filtered to key EU countries (DE, ES, FR, PT, NL) — even if impressions are near-zero, note what queries exist. Any EU-country impressions for this topic are valuable signal.

Every keyword sourced here comes with quantitative priority signals. The rest (Steps 2–4) fill in the gaps for keywords SEAR doesn't yet rank for.

### Step 2 — Seed expansion via web search

Broaden the seed topic into a full keyword universe. For each seed, run these searches and capture what comes up:

1. **Direct search**: search the seed term itself — note autosuggest variations and "People Also Ask" (PAA) questions in results
2. **Question variants**: search "how to [seed]", "what is [seed]", "why does [seed]", "best [seed]"
3. **Comparison searches**: "[seed] vs", "alternatives to [seed]", "is [seed] worth it"
4. **Forum/community language**: search "[seed] site:reddit.com" — the language people use in forums is gold for long-tail keywords
5. **Seasonal/contextual variants**: "[seed] for surfing", "[seed] for swimming", "[seed] for kids", "[seed] for triathletes"

Record every keyword variation — include PAA questions verbatim, they often match exactly what someone types into Google.

**Apply Non-negotiable #1 at this stage** — drop "custom earplugs" variants as they surface, and note them in a small "Filtered out (brand non-compliance)" list so Julia sees what was excluded and why.

### Step 3 — EU language keyword expansion (mandatory)

For every topic cluster identified in Steps 1–2, generate the native-language equivalents for each priority EU market. This is not optional — SEAR is expanding into Europe and zero EU-language content is a root cause of near-zero EU organic traffic.

For each major topic cluster (e.g. "surfer's ear prevention", "swim earplugs", "waterproof earplugs"), generate:

- 3–5 keyword variations per language per topic
- The search intent equivalent in that language (informational, commercial, transactional)
- Note whether a Google search in that language shows any content gap (are results thin or dominated by generic health sites?)

Use the reference terms from Non-negotiable #5 as starting points, then expand with topic-specific modifiers. For example, for "surfer's ear prevention" in German: "Surfer Ohr vorbeugen", "Gehörschutz beim Surfen", "Ohrenschutz Wellenreiten", "Exostose Ohr Surfer".

These keywords feed directly into Sheet 5 and the EU Market Language column in Sheet 1. Even though GSC won't have impression data for these yet, they are documented so `content-calendar-planner` can reserve EU-language content slots.

### Step 4 — Competitor content analysis (if competitor URL provided)

If the user gives a competitor URL, fetch it using WebFetch and analyse:
- Their blog/content topic clusters
- Page titles and H1s (these reveal their target keywords)
- What topics they cover that searplugs.com doesn't yet
- Any content gaps or thin pages that SEAR could do better

If no competitor URL is given, search for the top 2–3 competitors ranking for the seed term and fetch their blog/resource pages to identify their keyword strategy.

Focus on gaps and opportunities — what are they missing that a SEAR post could own? **Do not surface competitor brand queries as targets** (Non-negotiable #4); record them separately as "do not target" awareness.

### Step 5 — Classify and prioritise

For each keyword collected, assign:

| Field | Options |
|-------|---------|
| **Intent** | Informational / Commercial / Transactional |
| **Content type** | Blog post / Product page / FAQ / Landing page / Comparison page / EU landing page |
| **Priority** | High / Medium / Low |
| **Existing SEAR content?** | Yes / No / Partial |
| **Medical-adjacent?** | Yes / No |
| **EU Market Language** | DE / ES / FR / PT / NL / English (global) |
| **GSC impressions (90d)** | Number if SEAR already ranks; empty otherwise |
| **GSC position (90d avg)** | Number if SEAR already ranks; empty otherwise |
| **Source** | GSC / web search / forum / PAA / competitor / EU expansion |

**Intent classification guide:**
- **Informational**: "what is surfer's ear", "how to prevent swimmer's ear", "surfer's ear symptoms" — these want answers, not products. Best served by blog posts.
- **Commercial**: "best earplugs for surfing", "swim earplugs review", "surfer's ear prevention earplugs" — these are comparing options. Best served by blog posts with product CTAs, or comparison pages.
- **Transactional**: "buy surf earplugs", "SEAR earplugs", "earplugs for surfers" — these are ready to purchase. Best served by product/shop pages.

**Priority tiers (grounded in signals, not vibes):**
- **High**: One of the following is true — (a) GSC position 8–20 with ≥ 100 impressions in 90d (quick-win); (b) strong relevance to SEAR product-aligned primary keywords (surf earplugs, swim earplugs, surfer's/swimmer's ear prevention) AND low-to-medium SERP competition; (c) appears in autosuggest/PAA AND has clear informational value SEAR can own; (d) EU language keyword with clear search demand and no existing SEAR coverage (strategic EU expansion opportunity).
- **Medium**: Relevant but more peripheral. Higher SERP competition. OR GSC position > 20. OR EU language keyword with low confirmed demand.
- **Low**: Very niche, highly specific, or weak signals across the board.

### Step 6 — Check against existing SEAR content

Use the WordPress MCP (`wp_posts_search`) to check if searplugs.com already has content targeting any of the keywords found. Mark those as "Existing" to avoid duplication. Keywords with only partial coverage (a mention but no dedicated post) are "Partial" — often the best opportunities to develop further.

Cross-check with the GSC cannibalization data from Step 1 — if two SEAR URLs are already competing for the keyword, a new post is usually not the answer; flag for consolidation instead.

### Step 7 — Generate blog title suggestions

For the top 5–8 highest-priority informational and commercial keywords, write 1–2 compelling blog post title options. These should:
- Include the keyword naturally
- Be specific and promise something concrete (not generic)
- Match the brand voice — direct, confident, useful
- **Never** use "custom" / "custom-fit" / "bespoke" language

**Good title examples:**
- "Surfer's Ear: What Causes It, How Fast It Progresses, and How to Stop It"
- "The Best Earplugs for Open-Water Swimming (And Why a Secure Fit Matters)"
- "Swimmer's Ear vs Surfer's Ear: What's the Difference and How Do You Prevent Both?"
- "Can You Still Hear with Surf Earplugs? Why 9dB of Sound Reduction Matters"
- "Surf Earplugs for Hossegor: Why Cold-Water Surfers Need Ear Protection Year-Round" ← EU-targeted example

**Avoid:**
- Generic titles: "All About Swimmer's Ear", "Earplugs Guide"
- Keyword-stuffed titles: "Best Surf Earplugs for Surfer's Ear Swimmer's Ear Prevention"
- Any title with "custom" / "custom-fit" / "bespoke"

### Step 8 — Build the spreadsheet

Use the xlsx skill to produce a structured spreadsheet. The schema below is the canonical handoff format for `content-calendar-planner` and `seo-blog-writer`.

**Sheet 1: Keyword Research**
Columns: Keyword | Intent | Content Type | Priority | Existing on SEAR? | Medical-adjacent? | EU Market Language | GSC Impressions (90d) | GSC Position (90d) | Source | Notes

**Sheet 2: Blog Title Ideas**
Columns: Keyword | Suggested Title 1 | Suggested Title 2 | Notes

**Sheet 3: Competitor Gaps** (if competitor analysis was done)
Columns: Topic | Competitor URL | Gap Description | Suggested SEAR Angle

**Sheet 4: Filtered Out**
Columns: Keyword | Reason (brand non-compliance / competitor brand / medical over-claim / other)

**Sheet 5: EU Language Keywords** ← new mandatory sheet
Columns: Language | Market | Keyword (native language) | English equivalent | Intent | Priority | GSC Impressions | Notes

Sort Sheet 1 by Priority (High first), then GSC Impressions descending (where present), then Intent.
Sort Sheet 5 by Market (DE, ES, FR, PT, NL), then Priority.

Save the spreadsheet to the outputs folder with a clear filename: `keyword-research-[seed-topic]-[date].xlsx`

### Step 9 — Report back

After delivering the spreadsheet, give a short verbal summary:

```
✅ Keyword research complete: [seed topic]

**Grounded in signals:**
- [N] keywords sourced from GSC (where SEAR already has impressions)
- [N] keywords sourced from web search / PAA / forums
- [N] keywords filtered out (brand non-compliance / competitor / medical over-claim)
- [N] EU language keywords identified across [N] markets

**Quick wins (GSC position 8–20 + high impressions, no existing SEAR content):**
- [keyword] — pos [N], [N] impressions → [suggested content type]
- [keyword] — pos [N], [N] impressions → [suggested content type]

**EU language opportunities (top picks per market):**
- 🇩🇪 DE: [keyword] — [intent]
- 🇪🇸 ES: [keyword] — [intent]
- 🇫🇷 FR: [keyword] — [intent]
- 🇵🇹 PT: [keyword] — [intent]
- 🇳🇱 NL: [keyword] — [intent]

**Top blog title ideas:**
1. [title]
2. [title]
3. [title]

**Cannibalisation flags (check before new posts):**
- [query] — competing URLs: [URL A], [URL B]

**Spreadsheet saved:** [filename]
**Next step:** feed into `content-calendar-planner` or hand a specific keyword to `seo-blog-writer`. EU language keywords should be reserved as slots in the content calendar — at least 1 EU-targeted post per month.
```

---

## Things to avoid

- Don't fabricate search volume numbers — use GSC impressions where available, and directional signals (autosuggest, PAA, forums) otherwise. Be explicit about the source in the Source column.
- Don't include "custom earplugs" variants as target keywords — ever. Filter them out per Non-negotiable #1.
- Don't include keywords with no realistic path to ranking (e.g. "earplugs" as a standalone — dominated by Amazon and mass brands)
- Don't recommend keywords that would require medical claims SEAR can't substantiate — flag as `medical_adjacent: true` if adjacent, drop entirely if the keyword itself demands an over-claim
- Don't list competitor brands as target keywords — record them in "Do not target" awareness only
- Don't skip Sheet 5 (EU Language Keywords) — EU language research is mandatory per Non-negotiable #5

---

## Example invocations

**Invocation 1:**
> "Run keyword research on swimmer's ear"

**Invocation 2:**
> "What keywords should we be targeting for the surf earplugs product page?"

**Invocation 3:**
> "I want to see what [competitor URL] is ranking for and find the gaps we can go after"

**Invocation 4:**
> "Give me a deep dive keyword list for surfer's ear content — I want 40–50 keywords to build a content calendar from"

**Invocation 5:**
> "Use GSC data to find our best quick-win keywords right now — which positions are we 8–20 for with decent impressions?"

**Invocation 6:**
> "Find keyword opportunities for European surf markets — German, French, and Spanish"
