---
name: product-seo-optimizer
description: >
  Use this skill whenever the user wants to improve the SEO of WooCommerce product pages on searplugs.com. Triggers include: "optimize product pages", "improve product SEO", "rewrite product descriptions", "fix product meta", "product page SEO", "update product copy", "my product pages need work", or any request to review, audit, or rewrite WooCommerce product content. Also triggers when the user names a specific product and asks Claude to improve it. Always use this skill for product SEO tasks — do not attempt to rewrite product pages without it.
---

# Product SEO Optimizer — searplugs.com

This skill audits and rewrites WooCommerce product pages on searplugs.com for SEO performance and conversion. It works on a single product, a selection, or all products — and updates them directly in WordPress via the WooCommerce MCP.

This skill is part of the integrated searplugs.com SEO system defined in the **SEAR Plugs Automated SEO Strategy** doc. It is the canonical way to modify product page copy — so every change is logged to `ops_seo.actions` and flows into the weekly cadence. Findings from `seo-site-auditor` (e.g. brand-rule hits on product pages, missing meta fields) should be routed here to fix.

---

## Non-negotiables (read before every run)

These rules override anything else in this skill. Violating them breaks brand trust or creates compliance risk.

1. **SEAR earplugs are NOT custom, bespoke, or custom-moulded.** Never use "custom earplugs", "custom-fit earplugs", "custom-moulded earplugs", "bespoke earplugs", or "made-to-fit" — in titles, descriptions, tags, categories, meta fields, anchor text, or anywhere else. If the current product page uses these terms, removing them is a Critical fix (not optional). SEAR offers **waterproof earplugs** with a **3-tip + 4-wing sizing system** to find a secure fit without bespoke moulding. See the `product_sear_earplugs` auto-memory for the full product fact sheet.
2. **Medical-adjacency governance.** Product copy must not make clinical claims without backing — no "clinically proven", "medically approved", "cures surfer's ear", "prevents infection". Claim only what the product actually does: blocks water, maintains sound awareness via the 9dB acoustic filter, stays secure via the wing system. Hearing-protection and ear-health framing is fine; diagnosis/cure framing is not.
3. **No competitor brand names as targets.** Do not target or mention competitor brands (your category's named alternatives) in product copy. A neutral differentiation ("traditional plugs block 15dB and leave you isolated") is OK; named comparison in body copy is not.
4. **Pricing and status are off-limits.** Only touch copy and meta fields. Never change price, stock, visibility, or `status` via this skill.

---

## SEAR product fact sheet (use this, not general assumptions)

Source: product memory + B2B deck verified.

- **What they are:** Waterproof earplugs purpose-built for water sports — surfing, swimming, snorkelling, windsurfing, wingfoil, kitesurfing, bodyboarding, kayaking, wakeboarding.
- **Key differentiator:** 9dB acoustic filter vs ~15dB for competitors — blocks water/wind/cold while keeping full situational awareness. Tagline: **"Block Water. Embrace Sound."**
- **Sizing:** 3 earplug tip sizes (S/M/L) + 4 ear wing sizes (S/M/L/XL) — find a watertight seal without bespoke moulding
- **Wing system:** Ergonomic ear wings for a stable, secure fit — stays in during wipeouts
- **Materials:** Medical-grade silicone, hypoallergenic, ultra-lightweight, reusable
- **Accessories:** Optional leash with floatable element; waterproof magnetic case with carabiner
- **Price:** RRP £39.95 / €45.95
- **Correct primary keywords:** "waterproof earplugs for surfing", "waterproof earplugs for swimming", "earplugs for surfing", "swim earplugs", "surf earplugs", "earplugs for surfer's ear", "earplugs for swimmer's ear", "water sports earplugs"

---

## What this skill does

1. Fetches product content from WooCommerce
2. Audits each product against SEO and conversion best practices (with brand rules enforced)
3. Benchmarks against competitor product pages (for positioning insight, not for name-dropping)
4. Rewrites product titles, short descriptions, long descriptions, and Slim SEO meta fields
5. Updates the product directly in WordPress
6. Logs every change to `ops_seo.actions` so the weekly cadence picks it up
7. Reports every change made, before and after

---

## Inputs

**Required:**
- Scope: `all` / specific product name or ID

**Optional:**
- Primary keyword to target for each product (if not given, Claude infers from the correct-keyword list above)
- Focus area: `descriptions only` / `meta fields only` / `full rewrite` / `brand-rule fix only`

---

## What to audit and rewrite

### Product title
- Must include the primary keyword naturally
- Specific — avoid generic names like "Earplugs" or "Waterproof Plug"
- Under 70 characters (WooCommerce title SEO limit)
- Good format: `[Product Name] — [Key benefit or use case]`
- Good example: `SEAR Surf Earplugs — Waterproof Protection with Full Sound Awareness`
- Never: anything with "custom", "custom-fit", "bespoke"

### Short description (the excerpt shown near the Add to Cart button)
- 50–100 words
- Lead with the primary benefit, not the product name
- Include primary keyword in the first sentence
- 2–3 bullet points or a short punchy paragraph — not a wall of text
- Address the customer's core pain point (surfer's ear, swimmer's ear, water getting in, losing awareness)
- End with a confidence signal: the 9dB filter, medical-grade silicone, the 3+4 sizing system for a secure fit, or the RRP £39.95 price point

### Long description (main product page body)
- 300–600 words — enough to rank, not enough to overwhelm
- Structure:
  1. Opening paragraph — what this product is and who it's for (lead with water sport + awareness benefit)
  2. Key features section (H3 subheadings or bullet list — 9dB filter, wing system, sizing, material, accessories)
  3. How it works / what makes it different (the "Block Water. Embrace Sound." tension vs 15dB plugs that isolate you)
  4. Who it's for (use cases: surfing, swimming, open-water, triathlons, snorkelling, kitesurfing, parents of young swimmers, ENT-referred patients)
  5. Closing CTA paragraph
- Primary keyword in opening paragraph and at least one subheading
- Secondary keywords woven naturally (2–3 uses each)
- No filler phrases. Active voice. SEAR brand voice throughout (expert, warm, direct — never salesy)

### Slim SEO meta fields
- **Meta title**: under 60 characters, primary keyword near the front
  - Format: `[Product Name]: [Benefit] | SEAR`
  - Good example: `Surf Earplugs: Block Water, Embrace Sound | SEAR`
  - Good example: `Swim Earplugs — 9dB Acoustic Filter | SEAR`
  - Never: anything with "custom"
- **Meta description**: under 160 characters, keyword + specific benefit + implicit CTA
  - Good example: `Waterproof earplugs built for surfing. 9dB filter blocks water without muffling sound. Three tip sizes + four wings for a secure fit.`
  - Never: "custom-moulded", "custom-fit"

---

## Step-by-step workflow

### Step 1 — Fetch product data

Use `wc_products_search` to list all products (or `wc_get_product` for a specific one). For each product in scope, capture:
- Product ID, title, short description, long description
- Existing meta title and description (from Slim SEO fields)
- Categories, tags, price

If the long description is built with Elementor (common on this site), the raw content may contain shortcodes or builder markup. Work with whatever text is present — note if the description appears to be builder-constructed and flag it for Julia to review in the editor.

### Step 2 — Check upstream signals

Before rewriting:

- Check `ops_seo.rule_hits` for open findings on this product URL — these are the things that need fixing first (the audit already identified them)
- Check `ops_seo.actions` for recent edits — don't re-edit something that was just touched (cache lag risk)
- Pull `mcp__gsc__search_analytics` for the product URL to see which queries it actually shows up for — the rewrite should reinforce those winning queries, not drift away from them
- Pull `mcp__gsc__opportunity_finder` to find near-miss queries (position 8–20, high impressions) that the rewrite could lean into

### Step 3 — Benchmark competitor product pages (for positioning, not naming)

Search the primary keyword + "buy" or "shop" (e.g. "buy surf earplugs", "buy swim earplugs"). Fetch 2–3 competitor product pages using WebFetch and note:
- Their product title structure
- What benefits they lead with
- What they're missing (situational awareness, comfort during long sessions, wipeout stability, technical credibility)

Use this to ensure the SEAR rewrite is differentiated — but **never mention competitor names in the copy**. Frame differentiation around the product facts (e.g. "traditional plugs block around 15dB and leave you isolated — SEAR blocks water at 9dB so you can still hear your friends and the surf").

### Step 4 — Determine primary + secondary keywords

If the user provided keywords, use those — but still reject any that violate Non-negotiable #1 (e.g. if user says "custom earplugs for surfing", push back and use "surf earplugs" / "waterproof earplugs for surfing" instead).

Otherwise, infer from the correct-keyword list in the fact sheet:
- Primary: the most commercially valuable keyword for this specific product (e.g. "surf earplugs", "swim earplugs", "waterproof earplugs for surfing")
- Secondary (2–3): related phrases that describe use cases or benefits (e.g. "earplugs for surfer's ear", "water sports earplugs", "earplugs with sound awareness")

Confirm your keyword choice in the final report — if Julia disagrees, she can rerun with explicit keywords.

### Step 5 — Rewrite all copy

Apply the guidelines above to rewrite:
1. Product title (if it needs improvement)
2. Short description
3. Long description
4. Slim SEO meta title
5. Slim SEO meta description

Keep the SEAR brand voice throughout:
- Expert, warm, direct — like someone who surfs and genuinely cares about ear health
- Active voice. Sentences under 25 words.
- No filler: "In today's world", "It's important to note", "high quality", "state of the art"
- Specific and credible: "blocks cold water without muffling ambient sound" beats "great for water sports"
- Lead with the tension: water protection + sound awareness (the "Block Water. Embrace Sound." promise)

Write the full revised versions before updating anything in WordPress.

### Step 6 — Update the product in WordPress

Use `wc_update_product` to apply changes:
- `name`: updated product title (only if changed)
- `short_description`: rewritten short description (HTML OK — use `<p>` and `<ul>` tags)
- `description`: rewritten long description (HTML)
- Slim SEO meta fields: meta title and description

Make one update call per product. Confirm success from the API response.

If the product has tags with brand-rule violations (e.g. a "custom earplugs" tag), also:
- Remove the offending tag from this product via `wc_update_product`
- Flag the tag itself for cleanup in the final report (Julia can run `wc_update_product_tag` or delete the tag centrally)

### Step 7 — Log every change

Log each product update to `ops_seo.actions`:

```sql
INSERT INTO `YOUR_GCP_PROJECT.ops_seo.actions`
  (action_ts, action_type, url, detail)
VALUES
  (CURRENT_TIMESTAMP(), 'product_copy_update', '[product URL]',
   JSON '{"product_id":N,"fields_changed":["title","short_desc","long_desc","meta_title","meta_desc"],"primary_keyword":"...","brand_rule_fixes":N}');
```

If any `ops_seo.rule_hits` existed for this URL, also close them:

```sql
UPDATE `YOUR_GCP_PROJECT.ops_seo.rule_hits`
SET status = 'resolved', resolved_ts = CURRENT_TIMESTAMP()
WHERE url = '[product URL]' AND rule_id IN ('R_BRAND_CUSTOM','R_META_MISSING_TITLE',...) AND status = 'open';
```

Skip these logging steps silently if BigQuery is unreachable; note the skip in the final report.

### Step 8 — Report every change

For each product updated, report in this format:

```
✅ [Product Name] (ID: [ID])

TITLE
Before: [original]
After:  [new]

SHORT DESCRIPTION
Before: [original]
After:  [new]

LONG DESCRIPTION
Before: [summarise original in 1 line, e.g. "80-word generic description"]
After:  [summarise new in 1 line, e.g. "420-word benefit-led description with 3 H3 sections"]

META TITLE
Before: [original or "not set"]
After:  [new] ([N] chars)

META DESCRIPTION
Before: [original or "not set"]
After:  [new] ([N] chars)

KEYWORDS TARGETED: [primary] + [secondary 1], [secondary 2]

GOVERNANCE:
- Brand-rule fixes applied: [N] instances of "custom"/"bespoke" removed
- Medical claims reviewed: [pass/concerns noted]
- rule_hits closed: [N]
- Action logged to ops_seo.actions: [yes/no]
```

Then a one-line summary: "Updated [N] product(s). Key improvements: [2–3 bullet highlights]."

---

## Things to avoid

- Do not use "custom", "custom-fit", "custom-moulded", "bespoke", or "made-to-fit" anywhere — title, description, tags, or meta
- Do not delete or overwrite Elementor-built long descriptions without flagging it — complex builder layouts may need manual updating in the editor
- Do not make claims that can't be substantiated (e.g. "clinically proven", "used by 10,000 surfers", "FDA approved") unless this information was in the original content and verified
- Do not update `status`, pricing, or stock — only touch copy and meta fields
- Do not use superlatives without backing: "best earplugs in the world" → "9dB filter keeps you connected to sound while blocking water"
- Do not name or link to competitor brands in body copy
- Never publish changes without reporting them — always show before/after

---

## Example invocations

**Invocation 1:**
> "Optimise all the product pages for SEO"

**Invocation 2:**
> "Rewrite the product descriptions for the SEAR surf earplugs. Target keyword: waterproof earplugs for surfing."

**Invocation 3:**
> "Just fix the meta titles and descriptions on the product pages — they're not set."

**Invocation 4:**
> "Check product ID 42 and improve the short description — it's too vague."

**Invocation 5:**
> "The audit flagged brand-rule violations on every product page — sweep them all and remove any 'custom' language."
