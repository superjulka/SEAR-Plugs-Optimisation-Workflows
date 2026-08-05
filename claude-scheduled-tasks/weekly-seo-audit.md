---
name: weekly-seo-audit
schedule: "Mondays 07:00"
description: "Weekly: pull GSC signals, analyse keyword gaps & CTR opportunities, upsert rule_hits, email a prioritised report focused on organic traffic and sales"
---

> **Sanitised public version.** Replace every `[BRACKETED]` value with your own. The original runs against searplugs.com.

You are running the weekly SEO audit for `[YOUR-SITE]`. Today is Monday. Complete all steps below, then email `[YOUR-EMAIL]` the full report.

**Primary goal: identify what will drive more organic traffic, improve rankings, and convert visitors into customers. Brand compliance and medical/claims review checks are handled separately — do NOT include them in this report.** (That separation was a v2 lesson: v1 mixed compliance findings into the weekly email and drowned the traffic signals.)

---

## BigQuery credentials (required for Steps 3 & 4)

Set `GOOGLE_APPLICATION_CREDENTIALS` to your service-account key path (keep the key file OUT of any repo and out of task prompts — reference an environment variable or a local path only).

Use `client = bigquery.Client(project="[YOUR_GCP_PROJECT]", location="[YOUR_DATASET_LOCATION]")`.

Install if needed: `pip install google-cloud-bigquery db-dtypes --break-system-packages -q`

**BigQuery logging is MANDATORY. If credentials fail, stop and email a failure notice — do not continue with a skipped-logging audit.** (Also a v2 lesson: silent logging skips destroy the dedup memory.)

### ops_seo.rule_hits schema (exact column names)
- `url` STRING REQUIRED · `rule_id` STRING REQUIRED · `severity` STRING REQUIRED (CRITICAL/HIGH/MEDIUM/LOW)
- `evidence` STRING NULLABLE (evidence snippet + recommended fix)
- `first_seen_ts` / `last_seen_ts` TIMESTAMP REQUIRED · `status` STRING REQUIRED (open/resolved)
- `resolved_ts` TIMESTAMP NULLABLE · `resolved_by` STRING NULLABLE

Dedup key: `(url, rule_id)`. Insert new rows via `insert_rows_json`; update `last_seen_ts` on re-raised rows via DML UPDATE.

### ops_seo.actions schema
- `action_ts` TIMESTAMP REQUIRED · `action_type` STRING REQUIRED ("seo_audit") · `url` STRING NULLABLE · `detail` JSON NULLABLE

---

## Step 1 — Pull GSC signals (run all in parallel)

- `quick_wins` (pos 4–25, min 30 impressions, CTR ≤ 8%) — striking-distance keywords
- `opportunity_finder` (last 28 days) — rising impressions, low clicks, emerging queries
- `content_decay` (page dimension, min 100 impressions) — pages losing visibility
- `cannibalization` — pages competing for the same queries
- `device_country_breakdown` (breakdown: "country") — for the priority-markets section
- `position_tracking` (weekly, last 90 days) for your tracked keyword list `[YOUR ~10 CORE KEYWORDS]`

## Step 2 — Identify SEO improvement opportunities

For each issue ask: "will fixing this bring more qualified visitors, or convert more of the ones already landing?"

Check for: **A. Keyword gaps** (core keywords with near-zero impressions → create/strengthen/internally link) · **B. CTR underperformance** (good position, below-expected CTR — write the exact replacement meta copy) · **C. Ranking inconsistency** (wide week-to-week oscillation → content depth, internal links) · **D. Content decay** (refresh, update stats, add FAQ) · **E. Cannibalisation** (consolidate or redirect) · **F. Internal linking gaps** (every commercial post links to your shop page) · **G. Thin/missing meta** (direct CTR losses) · **H. Schema** (Product schema with price + availability on product pages).

## Step 3 — Dedupe against open rule_hits (REQUIRED)

Query existing open findings; mark each finding new or re-raised.

## Step 4 — Write findings to BigQuery (MANDATORY)

Upsert into `rule_hits`; log the run in `actions`; confirm counts in the email.

## Step 5 — Email the report

Plain text only, ASCII dividers. Sections, in order: **Keyword position trends** (4-week table per tracked keyword + 2–3 action bullets) · **Keyword gaps** · **CTR opportunities** (with exact replacement copy) · **Quick wins this week** (top 3, concrete) · **Content & ranking issues** (by severity) · **Priority markets** (mandatory every week, even when data is sparse — traffic table, anomalies with hypotheses, one concrete action, top content gap) · **Handoffs** (→ product optimizer, → blog writer, → needs human input, BigQuery logging status).

## Non-negotiables

- Every finding cites a specific URL and evidence — no generic advice
- Exact replacement copy for every meta recommendation
- Keyword-gap and priority-market sections are mandatory every week
- BigQuery logging is mandatory
- No brand-compliance findings in this report — handled separately
