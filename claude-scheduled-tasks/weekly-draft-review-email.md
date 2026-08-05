---
name: weekly-draft-review-email
schedule: "Fridays 09:00"
description: "Finds this week's WordPress drafts, verifies them live, and emails the owner edit links plus a publishing checklist — the human gate before anything goes live"
---

> **Sanitised public spec** (reconstructed from the live task).

You are the Friday review run — the human gate of the content pipeline.

1. **Find this week's drafts** — read the content calendar for rows set to `Draft in WP` this week.
2. **Verify each draft live in WordPress** — confirm the post exists, is still in draft status, and grab its edit URL. If WordPress is unreachable, fall back to calendar data and say so explicitly in the email — never present unverified drafts as verified.
3. **Email the owner** a plain-text summary: each draft with its edit link, a featured-image suggestion, and a short publishing checklist (read for accuracy, check the medical-review marker if present, confirm internal links, set featured image, publish).
4. **Create the email as a Gmail draft addressed to the owner.** Reliability lesson from the live system: automated *sending* (via browser automation) failed regularly; draft-plus-human-send is the dependable floor. If your email tooling supports direct sending and you trust it, send — but never let a send failure kill the run silently; always leave the draft behind.

Publishing remains a human decision. This task exists so that decision takes five minutes on a Friday instead of an evening of archaeology.
