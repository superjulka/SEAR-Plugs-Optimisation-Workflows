# B2B Outreach Playbook — Segment-First Drafting

> Generic, parameterised version of the workflow behind SEAR Plugs' UK wholesale campaign (236 prospects, 11 segments). Full story: [B2B Outreach at Founder Scale](https://julia-site-seven.vercel.app/workflows/b2b-outreach-drafts). Internal pricing and prospect details removed.

## The one rule that matters

**Claude drafts and analyses; the founder sends and decides.** Every outbound email is created as a Gmail draft. Nothing sends itself. This is a control decision, not a technical limitation.

## Parameters to set

- `[BRAND]`, `[PRODUCT]`, `[ONE-LINE DIFFERENTIATOR]`
- `[TRADE TERMS]` — trade price, RRP, minimum order (keep out of any public repo)
- `[SEGMENTS]` — your target segments, each with its own business model and problem framing
- `[RULES]` — e.g. no discounts without founder approval; no unreleased products; always personalise to the prospect's sport/region/role

## The message spine (every pitch follows it)

1. **Hook with their problem** — specific to the segment, referencing something real about the prospect
2. **Purpose-built solution** — what the product is, in their audience's terms
3. **Differentiation** — the one-line technical or commercial edge
4. **Traction proof** — honest, current, verifiable
5. **Specific ask** — stock it / recommend it / partner on it
6. **Low-friction CTA** — a sample offer or a single question, not "book a call"

## Segment personalisation (SEAR's real examples, as a pattern)

| Segment | Business model | Problem framing |
|---|---|---|
| Specialist retailers | Wholesale | Category gap on their shelf |
| Clinics / professionals | Wholesale + recommendation | Patient compliance, revenue per consultation |
| Schools / coaches | Kit recommendation | "Students can still hear coaching" |
| Clubs / communities | Discount-code advocacy (zero upfront cost) | Member benefit |
| Events / governing bodies | Sponsorship / goodie-bag | Audience experience, not wholesale |

Same product, five different commercial arguments. The segment table is the personalisation — write per-segment, not per-template.

## Process

1. **Research** each segment via web search; build the prospect list with named contacts where findable.
2. **Warm-check first**: before any cold pitch, search your inbox and sent folder for the prospect's domain. A prior customer or dropped thread means a threaded re-engagement reply, never a cold template.
3. **Draft in batches** (~5 per turn) via Gmail draft creation; return a summary table with send-priority notes.
4. **Day-7 follow-ups** for non-responders: 3–5 lines, re-open the conversation rather than re-pitch, threaded into the original conversation via threadId.
5. **Resume queue**: for long batch runs, generate all follow-up bodies into a JSON queue on disk first (`to, company, category, threadId, subject, body`), then create drafts batch by batch, updating the queue file *live* as each batch completes. Usage limits interrupt long runs; the queue makes them resumable — and updating it live (a lesson learned the hard way) keeps completion auditable.

## Honest expectations

Our verified numbers: ~7% human reply rate, 4 confirmed orders and 4 sample requests from 236 contacts. A ~93% silence rate is what cold B2B looks like, automated or not. The system's value is that researching, personalising and following up at that volume becomes feasible for one person — it does not make anyone reply.
