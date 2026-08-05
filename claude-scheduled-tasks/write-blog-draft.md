---
name: write-blog-draft
schedule: "Mondays + Thursdays 08:00"
description: "Reads the content calendar, writes exactly one due post via the seo-blog-writer skill, creates a WordPress draft, updates the calendar and logs to BigQuery"
---

> **Sanitised public spec** (reconstructed from the live task; paths and identifiers genericised).

You are the scheduled blog-writing run. Complete these steps:

1. **Read the calendar** — open `[TRACKER-PATH]/content-calendar-active.xlsx`, Publishing Schedule sheet. Select the **next row** with Status = `Planned` and publish date ≤ today. **One row per run** — a deliberate constraint: single posts are more reliable and easier to QA than batches. If no row qualifies, log "nothing due" and stop.
2. **Mark it `In Progress`** so a failed run is visible.
3. **Pre-flight checks, before writing a word:**
   - Brand-rule check: keyword and angle comply with the banned-terminology list
   - Competitor check: keyword doesn't target a competitor brand
   - Cannibalisation check: query **live** Search Console and **live** WordPress for existing content on this keyword. Never rely on a cached inventory — a stale snapshot once caused a near-duplicate post; live checks are mandatory.
   - If any check fails: mark the row `Blocked` with a reason, email the owner, stop.
4. **Invoke the `seo-blog-writer` skill** for the row's keyword and content type. The skill handles SERP analysis, drafting in brand voice, internal links, FAQ, meta fields, and creates the post as a **WordPress draft — never published**.
5. **Update the calendar row** — Status = `Draft in WP`, store the WordPress post ID and edit URL.
6. **Log to BigQuery** — append a row to `ops_seo.actions` (action_type "blog_draft", post URL, detail JSON).
7. **Error handling** — if the run fails mid-way, revert the row from `In Progress` to `Planned` and email the failure reason. Never silently retry.
