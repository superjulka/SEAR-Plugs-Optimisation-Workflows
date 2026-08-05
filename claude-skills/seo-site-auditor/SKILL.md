---
name: seo-site-auditor
description: >
  Use this skill whenever the user wants an SEO health check of searplugs.com — in full or for a specific content type. Triggers include: "SEO audit", "audit the site", "check my posts for SEO issues", "find SEO problems", "what needs fixing", "check product pages", "are my meta descriptions set", "look for thin content", "how is the site doing SEO-wise", or any request to review, analyse, or improve the SEO quality of existing content on the site. Also triggers for partial audits like "audit just the blog posts" or "check the product pages for keyword issues". Always use this skill for any SEO audit task — do not attempt an audit without it.
---

# SEO Site Auditor — searplugs.com

This skill runs a structured SEO audit of searplugs.com content — posts, pages, and/or WooCommerce products — and delivers a prioritised action report with quick wins clearly highlighted. The goal is to surface concrete, fixable issues rather than produce a generic checklist.

This skill is part of the integrated searplugs.com SEO system defined in the **SEAR Plugs Automated SEO Strategy** doc. Audit findings map 1:1 to the rules engine (`ops_seo.rule_hits`) where possible and feed into the weekly cadence review. Fixes applied through `product-seo-optimizer` or `seo-blog-writer` are tracked in `ops_seo.actions` — this skill reads from both to dedupe and prioritise.

---

## Non-negotiables (read before every run)

1. **Brand-rule check is Critical severity.** Any use of "custom earplugs", "custom-fit", "custom-moulded", "bespoke earplugs", or "made-to-fit" on any post, page, product, tag, or meta field must be flagged as a **Critical** issue. These terms make a claim SEAR cannot fulfil (no bespoke audiologist service) and attract the wrong audience. Correct language: "waterproof earplugs", "surf earplugs", "swim earplugs", "water sports earplugs".
2. **Medical-adjacency check is High severity.** Flag any unsupported clinical claim (dosage, prognosis, "cures", specific medical advice) on medical-adjacent posts (surfer's ear, swimmer's ear, treatment, surgery, hearing loss). These pages must link to reputable sources (NHS, Mayo Clinic, ENT UK) for definitive claims.
3. **Competitor-brand check.** Flag any content that names or links to competitor earplug brands (your category's named alternatives) as a High issue — unless the mention is clearly part of a differentiation/comparison context approved by Julia.
4. **Verify before flagging.** Every issue must be tied to a specific URL and a specific evidence snippet. Do not flag patterns you haven't verified.

---

## What this skill produces

- A prioritised issue list: Critical → High → Medium → Low
- Quick wins flagged (high impact, low effort)
- Specific recommended fixes per item — not vague advice
- An `.xlsx` audit report saved to the outputs folder
- Rows written back to `ops_seo.rule_hits` (when BigQuery is available) so the rule engine has a current view of open findings
- A brief verbal summary of the top 3–5 things to fix first

---

## Inputs

**Required:**
- Scope: choose one of: `all` / `posts` / `pages` / `products`

**Optional:**
- Focus area (e.g. "only check meta descriptions", "focus on thin content", "check heading structure", "brand-rule sweep")
- Specific post/page/product to audit (by title or ID)

If no scope is given, default to `all`.

---

## Checks to run

Run every applicable check below for each piece of content in scope. Not every check applies to every content type — use judgment (e.g. "internal links" is less relevant for a shop product page than for a blog post).

### Critical (fix immediately — brand risk or direct ranking harm)

| Check | What to look for |
|-------|-----------------|
| **Brand-rule violation** | Any use of "custom earplugs", "custom-fit", "custom-moulded", "bespoke", "made-to-fit" anywhere on the URL — body, headings, tags, meta fields, Slim SEO fields, anchor text. Flag the exact snippet and fix suggestion. |
| **Missing meta title** | Slim SEO meta title field is empty — Google will auto-generate one, often poorly |
| **Missing meta description** | Slim SEO meta description field is empty — kills click-through rate from search results |
| **No H1 tag** | Post/page has no H1, or the H1 doesn't contain a target keyword |
| **Duplicate meta title or H1** | Two or more posts/pages share the same title or H1 — causes keyword cannibalization. Cross-check with `mcp__gsc__cannibalization`. |
| **Broken internal links** | Links to other searplugs.com pages that return 404 |
| **Unsupported clinical claim on medical-adjacent content** | Dosage, prognosis, or "cure" claims without a reputable medical citation |

### High (fix soon — meaningful ranking or trust impact)

| Check | What to look for |
|-------|-----------------|
| **Thin content** | Post/page body under 600 words — Google may not index these well |
| **No internal links** | Blog post has zero links to other searplugs.com content |
| **Meta title too long** | Meta title over 60 characters (gets truncated in search results) |
| **Meta description too long** | Meta description over 160 characters (gets cut off) |
| **Missing target keyword in meta title** | The post's apparent target keyword doesn't appear in the meta title |
| **Keyword missing from H1** | The post's apparent target keyword is absent from the H1 |
| **No CTA to shop** | Blog post has no link to searplugs.com/shop or any product page |
| **Competitor brand mention** | Page names or links to a competitor earplug brand outside an approved differentiation context |
| **GSC indexing coverage issue** | URL flagged in `mcp__gsc__indexing_coverage` as Excluded, Crawl Anomaly, or Not Indexed |
| **GSC quick-win opportunity unmet** | URL surfaces in `mcp__gsc__quick_wins` (position 5–15, impressions ≥ N) but on-page optimisation is weak (e.g. primary keyword missing from H1 or intro) |

### Medium (worth fixing — incremental gains)

| Check | What to look for |
|-------|-----------------|
| **Weak heading structure** | Skipped heading levels (e.g. H1 → H3 with no H2), or only one H2 for a long post |
| **Missing FAQ section** | Blog posts without an FAQ section lose People Also Ask real estate |
| **Keyword only in H1** | Target keyword appears only once in the whole post — underoptimised |
| **No alt text on images** | Images with missing or empty alt attributes |
| **Post not categorised** | Blog post has no category assigned |
| **Fewer than 3 tags** | Posts with minimal tags reduce topical signal |
| **Tag hygiene** | Near-duplicate tags exist (e.g. "surfer's ear" + "surfers ear" + "surfer ear") — flag for consolidation |
| **No focus keyword set** | Useful if Slim SEO has a focus keyword field — check if set |
| **Cannibalisation risk** | Two SEAR URLs both ranking pages 1–10 for the same query in GSC (`mcp__gsc__cannibalization`) |

### Low (nice to have)

| Check | What to look for |
|-------|-----------------|
| **Short meta description** | Meta description under 120 characters — likely under-selling the page |
| **Generic anchor text** | Internal links using "click here" or "read more" instead of descriptive text |
| **Post not updated in 12+ months** | Content may be stale — flag for a refresh review (prioritise if `mcp__gsc__content_decay` shows the URL declining) |
| **Product missing long description** | WooCommerce product has only a short description — misses keyword real estate |

---

## Step-by-step workflow

### Step 1 — Pull the signal set

Before touching any content, gather the external signals that prioritise the audit:

- `mcp__gsc__quick_wins` — URLs where small changes could shift position 5–15 → top 3
- `mcp__gsc__cannibalization` — queries with multiple competing SEAR URLs
- `mcp__gsc__indexing_coverage` — URLs Google is struggling to index
- `mcp__gsc__content_decay` — URLs with declining impressions/clicks over 90 days
- `mcp__gsc__opportunity_finder` — high-impression, low-CTR URLs

Cache these results. Every audit finding should reference the relevant signal where applicable (e.g. "H1 missing primary keyword AND GSC shows this URL at position 9 with 1,200 impressions — quick-win candidate").

### Step 2 — Fetch all content in scope

Use the WordPress MCP to pull the content:

- **Posts**: `wp_posts_search` (status: published)
- **Pages**: `wp_pages_search` (status: published)
- **Products**: `wc_products_search`

For each item, capture: ID, title, URL/slug, word count (if available), categories, tags, and any meta field data returned.

If the scope is large (20+ items), process in batches to stay organised.

### Step 3 — Run checks per item

For each piece of content, work through the applicable checks from the table above. Where you can determine the answer from the WordPress MCP data alone (e.g. missing meta fields, no tags), do so. Where you need to see the rendered page (e.g. heading structure, internal links, image alt text, brand-rule text scan), use WebFetch on the live URL.

Be efficient: if the focus area is narrow (e.g. "only meta descriptions"), skip irrelevant checks rather than running everything. **Exception:** the brand-rule check (Critical) and medical-adjacency check (Critical/High) run on every audit regardless of focus area.

For each issue found, record:
- **Item**: post/page/product title
- **URL**: the live URL
- **Issue**: short description of the problem + evidence snippet (e.g. the exact phrase that violates the brand rule)
- **Severity**: Critical / High / Medium / Low
- **Rule ID**: mapped to `ops_seo.rule_hits` rule code (e.g. `R_BRAND_CUSTOM`, `R_META_MISSING`, `R_THIN_CONTENT`, `R_CANNIBAL`) — see Appendix
- **Recommended fix**: a specific, actionable instruction (not "add a meta description" — write what the meta description *should* say)
- **Quick win?**: Yes if the fix takes under 10 minutes and has High or Critical severity
- **GSC signal**: if applicable, the relevant GSC MCP signal backing the prioritisation

### Step 4 — Dedupe against open rule hits

If BigQuery is reachable, query `ops_seo.rule_hits` for open (unresolved) hits on the URLs in scope. Mark findings as `new` / `existing` / `re-raised`. This prevents the audit from re-flagging issues Julia has already triaged.

```sql
SELECT url, rule_id, first_seen_ts, status
FROM `YOUR_GCP_PROJECT.ops_seo.rule_hits`
WHERE status = 'open' AND url IN UNNEST(@scope_urls);
```

Also check `ops_seo.actions` for recent fix activity — e.g. if a meta description was just updated today, don't re-flag it as missing (there may be a cache lag).

### Step 5 — Identify patterns

After checking individual items, step back and look for site-wide patterns:
- Are meta descriptions missing across the board, or just some posts?
- Is thin content clustered in one category?
- Are products consistently missing long descriptions?
- Does the brand-rule violation appear in a template (e.g. all product page copy uses "custom earplugs")?

Patterns become the highest-priority recommendations because fixing them systematically is more valuable than one-off fixes.

### Step 6 — Build the audit report

Use the xlsx skill to produce a spreadsheet with these sheets:

**Sheet 1: Issue Log**
Columns: Item Title | URL | Content Type | Rule ID | Issue | Evidence | Severity | Quick Win? | Recommended Fix | GSC Signal | Dedupe Status

Sort by: Severity (Critical first), then Quick Win (Yes first), then Rule ID.

**Sheet 2: Summary Dashboard**
A simple counts table:
- Total items audited
- Issues by severity (Critical / High / Medium / Low counts)
- Quick wins count
- % of posts missing meta descriptions
- % of posts with thin content
- Brand-rule violations count (should be 0 in healthy state)
- Medical-adjacency issues count
- Top 3 recommended priorities

**Sheet 3: Quick Wins**
Just the rows from Sheet 1 where Quick Win = Yes, sorted by severity. This is the "start here" tab.

**Sheet 4: Patterns**
Site-wide patterns identified in Step 5, with a recommended bulk action for each.

Save the report to the outputs folder as: `seo-audit-[scope]-[date].xlsx`

### Step 7 — Write findings to `ops_seo.rule_hits`

When BigQuery is available, upsert each finding into `ops_seo.rule_hits`:

```sql
MERGE `YOUR_GCP_PROJECT.ops_seo.rule_hits` T
USING (SELECT @url AS url, @rule_id AS rule_id, @severity AS severity,
              @evidence AS evidence, CURRENT_TIMESTAMP() AS last_seen_ts) S
ON T.url = S.url AND T.rule_id = S.rule_id AND T.status = 'open'
WHEN MATCHED THEN
  UPDATE SET last_seen_ts = S.last_seen_ts, evidence = S.evidence
WHEN NOT MATCHED THEN
  INSERT (url, rule_id, severity, evidence, first_seen_ts, last_seen_ts, status)
  VALUES (S.url, S.rule_id, S.severity, S.evidence, CURRENT_TIMESTAMP(), S.last_seen_ts, 'open');
```

If `ops_seo.rule_hits` doesn't exist yet, skip this step silently and note it in the report.

### Step 8 — Log the audit run and report back

Append an audit-completion event to `ops_seo.actions`:

```sql
INSERT INTO `YOUR_GCP_PROJECT.ops_seo.actions`
  (action_ts, action_type, url, detail)
VALUES
  (CURRENT_TIMESTAMP(), 'audit_run', NULL,
   JSON '{"scope":"...","items_audited":N,"critical":N,"high":N,"medium":N,"low":N}');
```

After saving the spreadsheet, give a verbal summary:

```
✅ SEO audit complete — [N] items checked

**Top issues found:**
Critical: [N]  High: [N]  Medium: [N]  Low: [N]

**Brand-rule violations:** [N] (must be fixed — see Sheet 1 filter by rule R_BRAND_CUSTOM)
**Medical-adjacency issues:** [N]

**Start here (quick wins):**
1. [Specific fix] — affects [N] URLs
2. [Specific fix] — [item title]
3. [Specific fix] — [item title]

**Biggest pattern:**
[One sentence on the most common issue across the site]

**Handoffs:**
- Product page issues → hand to `product-seo-optimizer` with specific URLs
- Blog post issues → hand to `seo-blog-writer` (rewrites) or note in the next weekly review
- Rule hits written: [N] (or "skipped — BigQuery not reachable")

Full report: [filename]
```

---

## Appendix — Rule ID reference

Use these codes in the Rule ID column so findings map cleanly to `ops_seo.rule_hits`:

| Rule ID | Description | Default severity |
|---------|-------------|-----------------|
| `R_BRAND_CUSTOM` | "custom" / "bespoke" / "made-to-fit" language | Critical |
| `R_MED_CLAIM` | Unsupported clinical claim on medical-adjacent content | Critical |
| `R_META_MISSING_TITLE` | Slim SEO meta title empty | Critical |
| `R_META_MISSING_DESC` | Slim SEO meta description empty | Critical |
| `R_H1_MISSING` | No H1 or H1 lacks target keyword | Critical |
| `R_DUPE_META` | Duplicate meta title/H1 across URLs | Critical |
| `R_BROKEN_LINK` | Internal link returning 404 | Critical |
| `R_THIN_CONTENT` | Body under 600 words | High |
| `R_NO_INTERNAL_LINK` | Blog post with zero internal links | High |
| `R_META_LONG_TITLE` | Meta title > 60 chars | High |
| `R_META_LONG_DESC` | Meta description > 160 chars | High |
| `R_KW_NOT_IN_META` | Target keyword missing from meta title | High |
| `R_KW_NOT_IN_H1` | Target keyword missing from H1 | High |
| `R_NO_SHOP_CTA` | Blog post with no link to /shop or a product page | High |
| `R_COMPETITOR_MENTION` | Unapproved competitor brand mention | High |
| `R_GSC_INDEX_ISSUE` | URL flagged by GSC indexing_coverage | High |
| `R_GSC_QUICKWIN_UNMET` | GSC quick-win opportunity with weak on-page alignment | High |
| `R_HEADING_STRUCT` | Skipped heading levels / only one H2 | Medium |
| `R_NO_FAQ` | Blog post without an FAQ section | Medium |
| `R_KW_UNDEROPT` | Target keyword used only once | Medium |
| `R_ALT_MISSING` | Images without alt text | Medium |
| `R_NO_CATEGORY` | Post without a category | Medium |
| `R_TAG_SPARSE` | Fewer than 3 tags | Medium |
| `R_TAG_DUPE` | Near-duplicate tags | Medium |
| `R_CANNIBAL` | Two SEAR URLs competing for the same query | Medium |
| `R_META_SHORT_DESC` | Meta description under 120 chars | Low |
| `R_ANCHOR_GENERIC` | Generic anchor text (click here / read more) | Low |
| `R_STALE_CONTENT` | Post not updated in 12+ months | Low |
| `R_PRODUCT_SHORT_DESC_ONLY` | WooCommerce product missing long description | Low |

---

## Things to avoid

- Don't produce a generic "best practices" list — every finding must be tied to a specific post, page, or product on searplugs.com
- Don't recommend fixes that contradict the brand (no "add 'custom' to the meta title")
- Don't flag issues you haven't actually verified — if you couldn't fetch a page, say so rather than guessing
- Don't overwhelm with Low-severity issues if Critical/High items exist — prioritisation is the whole point
- Don't re-flag issues that already exist as open `rule_hits` without marking them as `existing` — Julia has already seen them

---

## Example invocations

**Invocation 1:**
> "Run a full SEO audit of the site"

**Invocation 2:**
> "Audit just the blog posts — I want to know which ones have thin content or missing meta descriptions"

**Invocation 3:**
> "Check the product pages for SEO issues"

**Invocation 4:**
> "Do a quick audit and just tell me the most important things to fix"

**Invocation 5:**
> "Sweep the whole site for the 'custom earplugs' brand-rule violation — I want every instance flagged"
