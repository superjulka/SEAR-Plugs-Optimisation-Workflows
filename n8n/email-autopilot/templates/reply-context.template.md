# Brand Context template (customer-safe)

Fill this in and paste it into the `BRAND CONTEXT` block of the `Build Draft Request` node's `DRAFT_SYS` prompt.

**Rule of thumb:** if you wouldn't be comfortable seeing a line quoted verbatim in an outbound customer email, it doesn't belong here. Keep confidential data (financials, contracts, wholesale pricing you don't want disclosed) OUT — anything here can surface in a draft.

```
ABOUT: {{What your business does, one or two lines. Tagline. Website URL.}}

PRODUCTS / SERVICES: {{The key facts a reply might need — what you sell, what it does, the differentiators. Keep it factual.}}

SIZING / OPTIONS / SPECS: {{If relevant — how customers choose the right variant, common questions.}}

PRICE: {{Public pricing only, e.g. "RRP $X on {{website}}". If you have wholesale/trade pricing you're happy for the AI to quote to genuine B2B, add it here; otherwise leave it out and let the guardrail set needs_founder_input=true for pricing topics.}}

POLICIES (leave a field blank/[EDIT] if unknown — the AI is told not to invent these):
- Shipping: {{regions + typical times}}
- Returns / exchange: {{window + how}}
- Warranty: {{...}}

FAQ (optional — paste your published Q&As so the AI can answer common questions):
Q: {{...}}  A: {{...}}
Q: {{...}}  A: {{...}}
```

## LIVE STATUS lives elsewhere

Don't hardcode current stock/promos/delays here — those change often and belong in the **Google Sheet** (`manual_status` row), so you can edit them in one place without touching the workflow. The draft prompt treats the Sheet's `manual_status` as authoritative.
