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

## What your assistant can do with it

| tool | what it does |
|---|---|
| `search_businesses` | find businesses in the index by query / city / vertical |
| `get_profile` | the full owner-verified profile (hours, services, facts, booking availability) |
| `get_reviews` | verified reviews — every one tied to a real completed booking |
| `check_availability` | open slots: opening hours ∖ existing bookings ∖ live calendar busy-times |
| `request_booking` | place a booking **request** — the business confirms every request |
| `check_booking` | status of a request by reference |
| `submit_review` | relay a customer's own review after a verified visit (agents never author sentiment) |

Booking is **request→confirm**: the business stays the authority, nothing is charged, and the
response always includes a reference + status URL to poll.

## Add it to your assistant

**claude.ai / Claude Desktop** — Settings → Connectors → *Add custom connector* → paste
`https://visiblefront.com/api/mcp`. Done.

**ChatGPT** — Settings → Connectors (requires developer mode where applicable) → add MCP server
with the same URL.

Anything else that speaks MCP over streamable HTTP works the same way — the server is
stateless and unauthenticated by design.

## From the command line (no MCP needed)

The underlying API is open, CORS-enabled, and self-documenting — every endpoint explains
itself on a bare GET:

```bash
# what can I do?
curl https://visiblefront.com/.well-known/ai-actions.json

# a business's availability (defaults: next 7 days, business-local time)
curl "https://visiblefront.com/api/businesses/birds-barbershop-austin/availability"

# how do I book? (the endpoint documents its own schema)
curl https://visiblefront.com/api/businesses/birds-barbershop-austin/request-booking

# place a booking request
curl -X POST https://visiblefront.com/api/businesses/birds-barbershop-austin/request-booking \
  -H 'content-type: application/json' \
  -d '{"requestedTime":"2026-07-21T15:00:00","customer":{"name":"Your Name","email":"you@example.com"}}'
```

## The honest rules this index runs on

- **Rankings and placement are never for sale.** Visibility scores come from what AI engines
  actually say, methodology published at [visiblefront.com/methodology](https://visiblefront.com/methodology).
- **Reviews are receipts, not opinions-at-large**: only customers with a completed booking
  through the rail can review; reviews are never editable or removable for money; owners get a
  response right, never a delete right.
- **Profiles are owner-verified** — unclaimed businesses appear in index collections but never
  get fabricated profile pages.
- **Aggregate-only data**: nothing about individual searchers ever leaves the platform.

## Links

[visiblefront.com](https://visiblefront.com) · [the AI Visibility Index](https://visiblefront.com/index) ·
[methodology](https://visiblefront.com/methodology) · [manifesto](https://visiblefront.com/manifesto) ·
hello@visiblefront.com

*The server implementation lives in the VisibleFront platform monorepo; this repo is its public
documentation and integration guide. VISIBLEFRONT LTD, Company No. 17288404 (England & Wales).*
