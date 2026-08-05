# AI Inbox Autopilot (n8n + Claude)

Turn your inbox into **reviewed drafts instead of a blank box.** Every incoming email is classified, enriched with real context (order data, the conversation so far, and your current operational status), and answered with an on-brand draft that lands in Gmail ready for a human to approve and send. **Nothing sends automatically** — the draft is the gate.

Built to run 24/7 on a cheap self-hosted [n8n](https://n8n.io) instance, using the [Claude API](https://console.anthropic.com) for the AI. Two small workflows, ~$5/month of hosting plus metered API usage.

> This is a sanitised, brand-agnostic version of a system originally built for a small e-commerce company run by two founders around full-time jobs — where slow email replies were the single biggest bottleneck. Everything specific has been replaced with `{{PLACEHOLDERS}}`. MIT licensed.

---

## Why this design

- **Human-in-the-loop by default.** The AI drafts and prioritises; a person sends and decides. Every reply is a Gmail *draft*. This is a control and brand-safety choice, not a limitation — it means you can put an LLM near customer email without fear.
- **Event-driven, not scheduled.** It runs the moment an email arrives, on an always-on server — not when your laptop happens to be open.
- **Cheap classify → expensive write.** A tiny model (Haiku) decides whether an email is even worth answering; the stronger model (Sonnet) only runs on genuine, actionable mail. Noise never reaches the expensive call.
- **Three layers of memory:**
  1. **Thread history** — reads the real Gmail conversation (source of truth), so the reply knows what you already told *this* person.
  2. **Operational memory** — a shared Google Sheet of "what's true right now" (stock, ETAs, delays, promos) that a second daily workflow keeps updated from your recent sent mail, so a fact you told 3 people this week reaches customer #4.
  3. **Brand context** — your customer-safe product/policy facts, embedded in the prompt.
- **Grounded, not hallucinated.** The prompt forbids stating facts that aren't in the provided context, and flags any draft that needs a human decision (pricing, contracts, anything sensitive).

---

## Architecture

```
WORKFLOW 1 — Email Triage & Drafter (trigger: new email)
  Gmail Trigger
    → Prepare Email            (robust sender/subject/body extraction)
    → Claude Classify [Haiku]  → Parse Classification
    → IF actionable? ── no ──→ Skip
         │ yes
    → Gmail: Get Thread → Summarise Thread History
    → Google Sheets: Read Ops Facts → Compose Ops Facts
    → IF needs order? ── yes ──→ Store lookup → order context
                        ── no ──→ empty context
    → Claude Draft [Sonnet]    → Parse Draft (flags human-review)
    → Gmail: Create Draft (threaded, never sends)
    → Get Labels → Resolve Label IDs → Add Category Labels

WORKFLOW 2 — Ops Facts Refresher (trigger: daily schedule)
  Schedule → Gmail: Recent Sent (7d) → Compact → Claude Extract [Haiku]
    → Parse Facts → Google Sheets: Write auto_facts
```

The two workflows are intentionally **separate** — the drafter is real-time; the refresher is a once-a-day batch job. They communicate only through the Google Sheet.

---

## What you need

- A **self-hosted n8n** instance reachable over **HTTPS** (Gmail OAuth requires it — see note below). A $5–6/month VPS is plenty.
- An **Anthropic API key** (console.anthropic.com) — billed separately from any Claude.ai subscription.
- A **Google/Gmail account** for the inbox (Google Workspace recommended — lets you make the OAuth app "Internal", avoiding verification and token-expiry headaches).
- A **Google Sheet** for the operational memory.
- *(Optional)* a **store integration** for order lookups. The example uses the WooCommerce node — swap it for Shopify/your platform, or delete that branch if you don't need order context.

> **HTTPS is mandatory for Gmail OAuth.** Google only accepts an `https://` redirect URI (or `localhost`) — a plain `http://IP:port` is rejected. Put n8n behind a reverse proxy (Caddy is two lines) or a tunnel before creating the Gmail credential.

---

## Setup

### 1. Create credentials in n8n
- **Gmail account** — Gmail OAuth2 (create an OAuth client in Google Cloud Console, enable the Gmail API, redirect URI `https://YOUR-N8N/rest/oauth2-credential/callback`).
- **Anthropic API (x-api-key)** — a *Header Auth* credential: header name `x-api-key`, value = your Anthropic key.
- **Google Sheets** — Google Sheets OAuth2 (enable the Sheets + Drive APIs; you can reuse the same OAuth client).
- *(Optional)* **Store API** — your e-commerce platform's credential.

### 2. Create the operational-memory Sheet
A sheet with a tab named `facts` and these rows:

| key | value | updated_at |
|---|---|---|
| manual_status | `{{Your current status — e.g. "In stock. No delays."}}` | |
| auto_facts | | |

You hand-edit `manual_status`; the refresher workflow fills `auto_facts`.

### 3. Create Gmail labels
Create the labels referenced in the *Resolve Label IDs* node, e.g. `AI/Customer`, `AI/B2B`, `AI/Investor`, `AI/Ops`, and `AI/P1`…`AI/P4`. Edit the `segMap` in that node to match your segments and label names.

### 4. Import both workflows
`Workflows → Import from File` for each JSON in `/workflows`. Then open each node showing a credential warning and select the credential you made. In the two Google Sheets nodes, pick your Sheet + `facts` tab from the dropdown (this fills the document ID).

### 5. Fill in your brand
This is the important part. Edit these placeholders:
- **`Build Classify Request`** node → the `CLASSIFY_SYS` prompt: your segments, intents, and routing rules.
- **`Build Draft Request`** node → the `DRAFT_SYS` prompt: your **voice**, your **guardrails/non-negotiables**, and the whole **`BRAND CONTEXT`** block (product/service facts, public pricing, policies, optional FAQ). Keep confidential data out — it can surface in a reply.
- **`Resolve Label IDs`** node → the `segMap`.
- **`Prepare Email`** node → the `noiseRe` list (senders to auto-ignore).

### 6. Test, then activate
Keep both workflows inactive. Send yourself a test email, run the drafter with **Execute Workflow**, and check your Drafts folder. Run the refresher once manually and check the Sheet's `auto_facts` cell fills. When happy, set both **Active**.

---

## The reusable gotchas (why this repo saves you a day)

These are the bugs that took real debugging — already fixed in the code:

- **`from` is an object, not a string.** Gmail nodes return the sender as `{value:[{address,name}]}` (and other shapes across versions). Naively stringifying it gives `[object Object]` and a broken "To" field. The `Prepare Email` node handles every shape.
- **Claude returns a `thinking` block first.** Newer models can return `content: [ {type:'thinking'}, {type:'text'} ]`. Reading `content[0].text` gives you empty output and a blank draft. Every parse node does `content.find(b => b.type === 'text')` instead.
- **Gmail OAuth needs HTTPS.** Covered above — it's the #1 setup blocker.
- **Output-format instruction goes last.** The JSON-only instruction is repeated at the very end of the draft prompt for reliable parsing.

---

## Placeholders reference

| Placeholder | Where | What to put |
|---|---|---|
| `{{BRAND_NAME}}` | both prompts | Your business name |
| `{{ONE_LINE_DESCRIPTION}}` | classify prompt | One line on what you do |
| `{{VOICE_GUIDELINES}}` | draft prompt | Tone/voice rules |
| `{{BRAND CONTEXT block}}` | draft prompt | Customer-safe facts, pricing, policies, FAQ |
| `{{SENSITIVE_TOPICS}}` | draft prompt | What must be human-gated |
| `segMap` | resolve-labels node | segment → Gmail label |
| `REPLACE_SHEET_ID` | both Sheets nodes | auto-filled when you pick the Sheet |
| `noiseRe` | prepare-email node | senders to ignore |

See `/templates` for a ready-to-fill brand-context file and the Sheet layout.

---

## Cost

- **Hosting:** ~$5–6/month for a small VPS.
- **API:** one cheap Haiku classification per email + one Sonnet draft only for actionable mail; the daily refresher is one small Haiku call. For a low-volume inbox this is typically a few dollars a month. Set a spend cap in the Anthropic console.

## Safety notes

- Never put confidential data (financials, contracts, wholesale pricing you don't want quoted) in the prompt or the Sheet — anything in context can appear in a draft.
- Keep the human gate. If you enable any auto-send, restrict it to the narrowest, fully-verifiable cases.
- Lock down your n8n instance (HTTPS, auth, firewall the raw port).

## License

MIT — see `LICENSE`. Adapt freely.
