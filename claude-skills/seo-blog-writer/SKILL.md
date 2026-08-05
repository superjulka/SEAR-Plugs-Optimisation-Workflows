---
name: seo-blog-writer
description: >
  Use this skill whenever the user wants to write, draft, create, or publish an SEO blog post for searplugs.com. Triggers include: any mention of "blog post", "write a post", "publish", "new article", "content for the site", or providing a keyword and asking Claude to write about it. Also triggers when the user says things like "write about surfer's ear" or "create a post on swimmer's ear prevention" — even if they don't say "SEO" or "blog" explicitly. This skill handles the full workflow: keyword research → competitor analysis → draft → WordPress draft creation.
---

# SEO Blog Writer — searplugs.com

This skill produces a fully researched, SEO-optimized blog post for searplugs.com and creates it as a draft in WordPress. It covers the complete workflow from keyword strategy through to publishing-ready content.

This skill is part of the integrated searplugs.com SEO system defined in the **SEAR Plugs Automated SEO Strategy** doc. It sits downstream of `keyword-researcher` and `content-calendar-planner`, and upstream of the weekly cadence review. Every published URL flows into the BigQuery data model (`dim_url`, `fact_url_perf`) so performance can be tracked over time.

---

## Non-negotiables (read before every run)

These rules override anything else in this skill. Violating them breaks brand trust or creates compliance risk.

1. **SEAR earplugs are NOT custom, bespoke, or custom-moulded.** Never use "custom earplugs", "custom-fit earplugs", "custom-moulded earplugs", "bespoke earplugs", or "made-to-fit" — in body copy, headings, anchor text, tags, meta fields, or CTAs. SEAR offers waterproof earplugs sized via a 3-tip + 4-wing system. Correct descriptors: "waterproof earplugs", "surf earplugs", "swim earplugs", "water sports earplugs". See the `product_sear_earplugs` auto-memory for the full product fact sheet.
2. **Medical-adjacency governance.** Posts covering medical symptoms, diagnoses, or treatment (surfer's ear surgery, otitis externa treatment, hearing-loss prognosis, etc.) must be created as `status: draft` only, flagged for human review, and must NOT make clinical claims beyond established medical consensus. No dosage, no prognosis specifics, no "cures". Link to reputable medical sources (NHS, Mayo Clinic, ENT UK) for definitive claims.
3. **No competitor brand names as targets or links.** Never target competitor brand keywords (your category's named alternatives) and never link to competitor product pages.
4. **Drafts only, never direct publish.** Always create posts with `status: draft`. Julia reviews before publishing.
5. **EU market awareness is mandatory.** SEAR now ships to Europe and EU organic growth is a top strategic priority. When a post covers a topic relevant to European water sports (surf, open-water swimming, triathlons, etc.), weave in references to European surf locations where natural (Hossegor, Peniche, Mundaka, Scheveningen, Sardinia, Zarautz). In product CTAs, mention that SEAR ships across Europe and include the EUR price point (€45.95) alongside the GBP price. This signals EU relevance to Google and removes the currency barrier for European readers.

---

## What this skill does

1. Check for existing keyword-researcher output or a calendar row and use it if available — otherwise run keyword research inline
2. Pull GSC + BigQuery signals to inform internal-link targeting and prioritisation
3. Analyse top-ranking competitor content to find gaps and angles
4. Write a brand-aligned, SEO-optimized post (1,200–2,000 words)
5. Create the post as a WordPress draft with all meta fields set
6. Log the publish action to `ops_seo.actions` so the weekly cadence can pick it up
7. Report back with a keyword and internal-linking summary

**Tip:** Running `keyword-researcher` on a topic before this skill gives richer secondary keyword data, PAA questions, and competitor gap analysis — all of which are picked up in Step 1 if a spreadsheet is present or if a calendar row is available.

---

## Inputs

**Required:**
- Primary keyword (e.g. "surfer's ear prevention", "best earplugs for swimming")

**Optional:**
- Secondary/related keywords
- Target audience angle (e.g. "aimed at parents of young swimmers")
- EU market focus (e.g. "target French surf audience", "optimise for German swimmers")
- Publish date (if scheduling)
- Tone notes (defaults to SEAR brand voice)

---

## Step-by-step workflow

### Step 1 — Keyword & SERP research

Before writing a single word, build a solid keyword foundation. How thorough this step needs to be depends on what the user has already done.

#### 1a — Check for existing keyword research or a calendar row

Look for upstream artifacts in this order:

1. **Content calendar** at `content-calendar-active.xlsx` in the outputs folder (the handoff file from `content-calendar-planner`). If a row matches the topic where Status = `Planned` and Publish Date ≤ today, use it — it already has primary/secondary keywords, audience, and a suggested title. Check the EU Market column — if it's populated, treat EU relevance as a requirement for this post.
2. **keyword-researcher spreadsheet** matching the topic, named like `keyword-research-[topic]-[date].xlsx`. If found:
   - Sheet 1 (Keyword Research) → secondary keyword priorities; check EU Market Language column for non-English equivalents to weave in or target with hreflang notes
   - Sheet 2 (Blog Title Ideas) → H1 inspiration
   - Sheet 3 (Competitor Gaps) → sharpen the content angle
   - Skip to Step 1c
3. **GSC MCP signals** for prioritisation, when no upstream artifact exists:
   - `mcp__gsc__opportunity_finder` — keywords where searplugs.com impressions are high but position is 8–20 (quick-win targets to weave in as secondaries)
   - `mcp__gsc__cannibalization` — check the primary keyword isn't already being competed for by another SEAR URL; if it is, flag and suggest a complementary angle instead of a new post
   - `mcp__gsc__position_tracking` on the primary keyword for baseline context

If no upstream research is found, run Steps 1b and 1c in full.

#### 1b — Run keyword research (if not already done)

Use web search to map the keyword landscape:

1. **Direct SERP check**: search the primary keyword — note autosuggest variations and PAA questions verbatim (these become your FAQ questions)
2. **Question variants**: "how to [topic]", "what is [topic]", "why does [topic]"
3. **Forum language**: "[topic] site:reddit.com" — captures real phrasing, matches long-tail queries
4. **Comparison searches**: "[topic] vs", "best [topic]", "[topic] for surfing/swimming"

Also check WordPress for existing SEAR content on this topic using `wp_posts_search`. If a post already covers this keyword closely, flag it rather than creating a duplicate — suggest a complementary angle. Cross-check with `mcp__gsc__cannibalization` for extra confidence.

#### 1c — Competitor page analysis

Fetch 2–3 of the top-ranking pages for the primary keyword using WebFetch. For each, note:
- Heading structure (H1/H2s) — reveals what sub-topics Google considers relevant
- Content gaps — what angle or depth are they missing?
- Word count estimate — sets the benchmark to beat

**Outcome**: A clear primary keyword, 3–5 secondary keywords, PAA questions for the FAQ, and a differentiating content angle.

---

### Step 2 — Plan the post structure

Before writing, map out the post structure. Think about it as building a brief:

- **Primary keyword**: exact phrase to target in H1
- **Secondary keywords**: 2–4 phrases to weave naturally into H2s and body
- **Content angle**: what makes this post different from what's already ranking?
- **H1**: must contain the primary keyword; should be compelling, not just the keyword verbatim
- **H2s** (4–6): keyword-rich subheadings that together cover the topic fully
- **FAQ section**: 3–5 questions pulled from People Also Ask or forum discussions
- **Internal links**: find 2+ existing searplugs.com posts or pages to link to. Use this order:
  1. If keyword-researcher was run, use the "Existing on SEAR?" column in Sheet 1
  2. Otherwise query `dim_url` in BigQuery (or `wp_posts_search`) for posts that match secondary keywords — prefer URLs that rank pages 8–20 on GSC for those keywords (they benefit most from an internal-link boost)
  3. Always include at least one link to `https://searplugs.com/shop` as a commercial handoff
- **EU relevance check**: does the topic apply to European surf or swim culture? If yes, plan to reference at least one European location naturally within the body (e.g. Hossegor, Peniche, Mundaka, Scheveningen, Sardinia, Zarautz). This signals geographic breadth to Google and improves EU relevance.
- **Medical-adjacency check**: if the post covers symptoms, diagnoses, or treatment, note this now — it affects the marker in Step 5

---

### Step 3 — Write the full post

Write 1,200–2,000 words following this exact structure:

#### Required structure:

```
[H1 — primary keyword included]

[Intro — 3 short paragraphs]
- Hook: a vivid, relatable opening (a scenario, a stat, a question)
- Problem: what's at stake if this is ignored
- Promise: what the reader will learn / get from reading this

[H2 — Section 1]
[body]

[H2 — Section 2]
[body]

[H2 — Section 3]
[body]

[H2 — Section 4]
[body]

(Add H2 — Section 5 or 6 if the topic warrants it)

[H2 — Frequently Asked Questions]

[FAQ item 1: Q + A]
[FAQ item 2: Q + A]
[FAQ item 3: Q + A]
(3–5 FAQs total)

[H2 — Protect Your Ears Before It's Too Late]  ← or similar CTA heading

[Closing paragraph + CTA]
```

#### SEAR brand voice rules (apply every time):
- Write like a surfer who genuinely cares about ear health — expert, warm, direct
- Active voice throughout. Passive voice is a red flag.
- Sentences under 25 words. If a sentence runs long, split it.
- No filler phrases: "In today's world", "It's important to note", "At the end of the day", "With that in mind" — cut them on sight
- Not salesy. Earn trust, then let the product speak for itself
- Not overly clinical. Use plain language for medical concepts, then link to reputable sources (NHS, Mayo Clinic, ENT UK) for detail
- Specific beats vague. "Cold water triggers bone growth in the ear canal" beats "water exposure can affect your ears"

#### SEO writing rules:
- Primary keyword appears in: H1, first 100 words of intro, at least one H2, and naturally 3–5x throughout
- Secondary keywords woven in naturally — never stuffed
- Each H2 section should be self-contained enough that a reader could skim to it and get value
- FAQ answers should be 40–80 words — long enough to be useful, short enough to potentially surface as a featured snippet
- Internal links: use descriptive anchor text. Not "click here" — use the topic of the page being linked to (e.g. "surfer's ear symptoms and treatment")
- CTA must link to https://searplugs.com/shop with anchor text that matches actual SEAR product language. Good: "SEAR surf earplugs", "SEAR's waterproof earplugs for surfing", "the SEAR earplug range". Never: "custom earplugs", "custom-fit earplugs", "bespoke earplugs".

#### EU market guidance (apply when the topic is relevant to European water sports):
- Mention at least one European surf or swim location naturally in the body copy — frame it as universal context, not a diversion. Example: "Whether you're paddling out at Hossegor or your local beach, cold water is cold water."
- In the product CTA, note that SEAR ships across Europe. Include EUR pricing alongside GBP: "Available for £39.95 / €45.95 with shipping across the UK and Europe." This removes the currency guessing barrier for EU readers and signals EU relevance to Google.
- If writing for a predominantly UK or EU surf audience, reference the relevant regional surf culture naturally — Hossegor for France, Peniche/Nazaré for Portugal, Mundaka/Zarautz for Spain, Scheveningen for the Netherlands, Lacanau for France.
- Do NOT force EU references into posts where they feel unnatural — relevance is the test. A post about surfer's ear in cold water? Hossegor fits. A post specifically about UK surf spots? No need to add EU references.

---

### Step 4 — Set Slim SEO meta fields

Write these before creating the WordPress post:

- **Meta title**: under 60 characters, includes primary keyword near the front
- **Meta description**: under 160 characters, includes keyword + a compelling reason to click

**Example format:**
```
Meta title: Surfer's Ear Prevention: What Actually Works | SEAR
Meta description: Cold water triggers bone growth in your ear canal. Here's how to prevent surfer's ear and protect your hearing long-term.
```

---

### Step 5 — Create the WordPress draft

Use `wp_add_post` to create the post. **Status is always `draft`** — no exceptions, and medical-adjacent content doubly so.

```
title: [H1 text]
content: [full post HTML]
status: draft
categories: [appropriate category — e.g. "Ear Health", "Surfing Tips"]
tags: [3–6 relevant tags — e.g. "surfer's ear", "exostosis", "ear protection", "surf earplugs", "swim earplugs"]
meta:
  slim_seo:
    title: [meta title]
    description: [meta description]
```

Tag rules:
- Use only tags that comply with the non-negotiables (no "custom earplugs", no competitor brand tags)
- Prefer consolidating tag taxonomy — check `wp_list_tags` and re-use existing tags rather than creating near-duplicates

If categories or tags don't exist yet, create them using `wp_add_category` or `wp_add_tag` before adding the post.

**Medical-adjacency flag:** if the post covers symptoms, diagnoses, or treatment, prepend a draft-only HTML comment so Julia spots it in the WordPress editor:

```
<!-- MEDICAL REVIEW NEEDED: clinical content — do not publish without human review -->
```

---

### Step 6 — Log the action and report back

**Log to `ops_seo.actions`** (when BigQuery is available) so the weekly cadence captures the publish event:

```sql
INSERT INTO `YOUR_GCP_PROJECT.ops_seo.actions`
  (action_ts, action_type, url, detail)
VALUES
  (CURRENT_TIMESTAMP(), 'blog_draft_created', '[post URL]',
   JSON '{"primary_keyword":"...","secondary_keywords":["..."],"medical_adjacent":false,"eu_targeted":false}');
```

Set `"eu_targeted": true` if EU locations or EUR pricing were included in the post.

If the `ops_seo.actions` table doesn't exist yet or BigQuery is unreachable, skip this step silently and note it in the report.

Then give the user a clean summary:

```
✅ Draft created: "[Post Title]"

**Keyword strategy:**
- Primary: [keyword]
- Secondary: [keywords]
- Content angle: [1 sentence on the differentiating angle]

**EU relevance:**
- EU locations referenced: [yes/no — which ones]
- EUR pricing included in CTA: [yes/no]

**Internal links used:**
- [anchor text] → [URL]
- [anchor text] → [URL]

**Slim SEO fields:**
- Title: [meta title]
- Description: [meta description]

**Governance flags:**
- Medical-adjacency: [yes/no — if yes, flagged in post for human review]
- Cannibalization check: [passed / potential overlap with URL X — flagged]
- Action logged to ops_seo.actions: [yes/no]

**Next step:** Review the draft in WordPress, add a featured image, and publish when ready.
```

---

## Things to avoid

- Do not use "custom", "custom-fit", "custom-moulded", "bespoke", or "made-to-fit" anywhere — body, headings, anchor text, tags, or meta (see Non-negotiables)
- Do not link to or name competitor earplug brands (your category's named alternatives)
- Do not make clinical claims without basis (stick to established medical consensus on surfer's ear / swimmer's ear)
- Do not pad the post to hit word count — every section should earn its place
- Do not create the post with `status: publish` under any circumstance
- Do not use stock H2s like "Introduction" or "Conclusion" — every heading should be informative
- Do not skip the cannibalization check — two SEAR URLs competing for the same keyword is worse than one good one
- Do not force EU references into posts where they feel unnatural — only add them where they genuinely fit

---

## Example invocations

**Invocation 1:**
> "Write a blog post targeting the keyword 'how to prevent swimmer's ear'"

**Invocation 2:**
> "Can you write a post about surfer's ear exostosis for ENT-referred patients?"

**Invocation 3:**
> "I want a new blog post. The keyword is 'best earplugs for open water swimming'. Secondary keywords: earplugs for triathlon, swimming ear protection."

**Invocation 4:**
> "Write a post about surf earplugs for the French market — target Hossegor surfers"
