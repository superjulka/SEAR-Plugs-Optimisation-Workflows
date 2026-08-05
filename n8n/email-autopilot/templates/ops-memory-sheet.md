# Operational-memory Google Sheet

A single Google Sheet is the shared "what's true right now" store. The drafter **reads** it on every reply; the refresher workflow **writes** the `auto_facts` row once a day.

## Layout

Create a spreadsheet, rename the first tab to **`facts`**, and set it up exactly like this (headers in row 1):

| key | value | updated_at |
|---|---|---|
| manual_status | In stock. No promotions. No known delays. | |
| auto_facts | | |

## How the two rows are used

- **`manual_status`** — you (and your team) edit this by hand. It is **authoritative**: the draft prompt is told never to contradict it. Change it whenever something changes globally, e.g. `Out of stock — restock ETA 10 Aug`. Every draft will then reflect it.
- **`auto_facts`** — the *Ops Facts Refresher* workflow fills this once a day by scanning your last 7 days of **sent** mail and extracting only general facts (stock, ETAs, delays, promos), stripping anything customer-specific. Treated as supporting context: if a draft relies on it, that draft is flagged for human review.

## Why a Sheet (vs. a database or a file)

- Visible and hand-editable by non-technical teammates.
- No extra infrastructure — you already have Google.
- Doubles as the single place to set "current status", so it never drifts from what the AI says.

## Wiring

In both workflows' Google Sheets nodes, pick this spreadsheet and the `facts` tab from the dropdown after import (n8n fills the document ID automatically).
