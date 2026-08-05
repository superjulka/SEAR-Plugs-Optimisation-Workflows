---
name: monthly-content-planning
schedule: "08:00 on the 1st of the month (set your own cadence)"
description: "Runs keyword research on core topics, then plans the coming month's content calendar — the input the write-blog-draft task consumes"
---

> **Sanitised public spec** (reconstructed from the live task).

You are the scheduled content-planning run. Complete these steps:

1. **Run the `keyword-researcher` skill** on the core topic list `[YOUR SEED TOPICS]`. It grounds everything in Search Console reality (current impressions, positions, cannibalisation risks), expands seeds via autosuggest/People-Also-Ask/forum language, runs the priority-market language expansion, and filters out banned and competitor-brand keywords. Output: a multi-sheet keyword spreadsheet, including an explicit "Filtered Out" sheet so exclusions are auditable.
2. **Run the `content-calendar-planner` skill** for the coming month. It combines the fresh keyword research, the current WordPress inventory, and Search Console quick-wins; applies seasonality and content-mix rules (~60% informational / 40% commercial, audience rotation, no clickbait title formulas); and assigns posts to the publishing slots (default: Monday + Thursday).
3. **Save `content-calendar-active.xlsx`** to `[TRACKER-PATH]` — Sheets: Publishing Schedule / Keyword Pool / Archive / Excluded. This file is the single source of truth the writing task reads; the human can pause or reorder anything by editing Status cells.
4. **Log the planning run** to `ops_seo.actions`.
5. **Email a summary** — the month at a glance, which slots are medical-adjacent (max 1/month, flagged for review), which slots target priority markets (min 1/month).

Encoded rules: never plan a keyword the researcher excluded; max one medical-adjacent slot per month; at least one priority-market slot per month; no "Complete Guide"/"Ultimate Guide" titles.
