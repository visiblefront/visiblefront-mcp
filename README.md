# VisibleFront MCP — book real local businesses from any AI assistant

**VisibleFront is an AI visibility platform that helps local businesses become recommended by
ChatGPT, Gemini and Perplexity through visibility monitoring, structured business profiles and
data optimization.** This repo documents the **VisibleFront MCP server** — one connector that
makes the whole VisibleFront index searchable, availability-aware, and bookable from any
MCP-speaking assistant.

```
Connector URL:  https://visiblefront.com/api/mcp
Transport:      Streamable HTTP (JSON-RPC 2.0) · stateless · no auth
Discovery:      https://visiblefront.com/.well-known/ai-actions.json
```

## Breaking change (2026-07-19)

`submit_review` now requires a `consent` field — a boolean, strictly `true` (the string
`"true"` is rejected). Missing or `false` returns `consent_missing`. Set it only after your
human has explicitly agreed to their review being published publicly under their first name +
last initial — ask, never assume. Consent is recorded. Callers built against an earlier version
of this README will get `consent_missing` until they add the field.

## What your assistant can do with it

| tool | what it does |
|---|---|
| `search_businesses` | find businesses in the index by query / city / vertical |
| `get_profile` | the full owner-verified profile (hours, services, facts, booking availability) |
| `get_reviews` | verified reviews — every one tied to a real completed booking |
| `check_availability` | open slots: opening hours ∖ existing bookings ∖ live calendar busy-times |
| `request_booking` | place a booking **request** — the business confirms every request |
| `check_booking` | status of a request by reference |
| `submit_review` | relay a customer's own review after a verified visit, with the human's explicit, recorded consent to publish (agents never author sentiment) |
| `list_markets` | coverage discovery — the countries, cities and verticals the index covers, with freshness and a published / data_only status per market |
| `get_collection` | the ranked collection for one market — the same JSON the public collection page serves (scores, per-engine signals, `/b` links, bookable flags) |

Booking is **request→confirm**: the business stays the authority, nothing is charged, and the
response always includes a reference + status URL to poll.

## Add it to your assistant

**claude.ai / Claude Desktop** — Settings → Connectors → *Add custom connector* → paste
`https://visiblefront.com/api/mcp`. Done.

**ChatGPT** supports MCP servers in developer mode; add the same URL where custom MCP servers
are configured.

Anything else that speaks MCP over streamable HTTP works the same way — the server is
stateless and unauthenticated by design.

Full how-to: [visiblefront.com/connect](https://visiblefront.com/connect)

## From the command line (no MCP needed)

The underlying API is open, CORS-enabled, and self-documenting — every endpoint explains
itself on a bare GET:

```bash
# what can I do? (REST discovery)
curl https://visiblefront.com/.well-known/ai-actions.json

# what can I do? (MCP JSON-RPC — same server the connector URL speaks to)
curl -X POST https://visiblefront.com/api/mcp \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'

# a business's availability (defaults: next 7 days, business-local time; window capped at 31 days)
curl "https://visiblefront.com/api/businesses/birds-barbershop-austin/availability"

# how do I book? (the endpoint documents its own schema)
curl https://visiblefront.com/api/businesses/birds-barbershop-austin/request-booking

# place a booking request
curl -X POST https://visiblefront.com/api/businesses/birds-barbershop-austin/request-booking \
  -H 'content-type: application/json' \
  -d '{"requestedTime":"2026-07-21T15:00:00","customer":{"name":"Your Name","email":"you@example.com"}}'
```

## Rate limits

Per IP, per tool: 120 requests/hour for read tools (`search_businesses`, `get_profile`,
`get_reviews`, `check_availability`, `check_booking`, `list_markets`, `get_collection`), 10/hour
for write tools (`request_booking`, `submit_review`). `check_availability` windows are capped at
31 days; `calendar_checked: false` in a response means the business's live calendar wasn't
consulted for that check, so the returned slots are opening-hours-only.

## The honest rules this index runs on

- **Rankings and placement are never for sale.** Visibility scores come from what AI engines
  actually say, methodology published at [visiblefront.com/methodology](https://visiblefront.com/methodology).
- **Reviews are receipts, not opinions-at-large**: only customers with a completed booking
  through the rail can review; reviews are never editable or removable for money; owners get a
  response right, never a delete right.
- **Profiles are owner-verified** — unclaimed businesses appear in index collections but never
  get fabricated profile pages.
- **Aggregate-only data**: every MCP call is logged as a sanitized demand event (tool, sanitized
  query/city/vertical, slug, outcome) — no customer fields, no review content, no raw IPs. Raw
  events age out at 90 days into monthly aggregates; only the aggregate ever leaves the platform.
  Demand never influences ranking, and placement is never for sale.

## The flywheel this connector powers

Every booking placed through the rail can become a **verified review**: after a completed visit
the customer gets a time-limited review link — real customers, real appointments, explicit
recorded consent, and never purchasable or editable by anyone, at any price. Verified reviews
make a profile more trustworthy — to assistants and to the index — so recommended businesses get
booked more, and booked businesses become more recommendable. Visibility → bookings → verified
reviews → more visibility. Businesses pay **0% of their bookings, ever**, so there is no fee to
route around — which keeps bookings on the rail, the loop honest, and the data clean for every
agent that reads it.

## Links

[visiblefront.com](https://visiblefront.com) · [how to connect](https://visiblefront.com/connect) ·
[the AI Visibility Index](https://visiblefront.com/index) ·
[methodology](https://visiblefront.com/methodology) · [manifesto](https://visiblefront.com/manifesto) ·
hello@visiblefront.com

*The server implementation lives in the VisibleFront platform monorepo; this repo is its public
documentation and integration guide. VISIBLEFRONT LTD, Company No. 17288404 (England & Wales).*
