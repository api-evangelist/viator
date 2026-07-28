---
name: Ingest and maintain the Viator product catalogue
description: >-
  Build and keep current a local mirror of Viator's bookable product catalogue and its availability
  schedules using the bulk and modified-since endpoints, then resolve the opaque tag, destination
  and location references the product records carry.
api: openapi/viator-partner-api-v2-openapi.json
generated: '2026-07-28'
method: generated
source: >-
  https://docs.viator.com/partner-api/technical/ (Workflows - Ingesting and updating the product
  catalogue, Resolving references, Ingesting and updating availability schedules) plus verified
  operationIds from openapi/viator-partner-api-v2-openapi.json
operations:
  - productsModifiedSince
  - productsBulk
  - products
  - productsTags
  - destinations
  - locationsBulk
  - availabilitySchedulesModifiedSince
  - availabilitySchedulesBulk
  - availabilitySchedules
tier: Full-access Affiliate or above
---

# Ingest and maintain the Viator product catalogue

Viator expects partners to hold a **local mirror** of the catalogue and query it locally, not to
call through on the read path. Every large collection is exposed three ways — single fetch, bulk
fetch by identifier list, and a cursored `modified-since` delta feed.

## Before you start

- Send `Accept: application/json;version=2.0` on **every** request. Omitting the version returns
  HTTP 400 `INVALID_HEADER_VALUE`.
- Send your organisation key in the `exp-api-key` header.
- Send `Accept-Language` on any call that returns natural-language fields.
- Do all development against `https://api.sandbox.viator.com/partner`.
- `productsModifiedSince` and `productsBulk` are **not** available to Basic-access affiliates.

## Initial load

1. Call `productsModifiedSince` with `count` and **no** `modified-since` and **no** `cursor`. This
   starts the feed at the beginning of the catalogue.
2. Persist every product record keyed on `productCode`, and persist the `nextCursor` returned with
   the page.
3. Repeat with `cursor` set to the previous `nextCursor` until the feed is exhausted.
4. Store the final cursor. It — not a wall-clock timestamp — is your ingestion watermark.

Do not paginate this feed with `modified-since` on subsequent pages; the cursor is the only reliable
continuation token.

## Incremental updates

1. Call `productsModifiedSince` with the stored `cursor`.
2. Apply each returned record over your local copy, keyed on `productCode`.
3. Save the new `nextCursor`.

Choose a cadence, not a call-per-request pattern. If a product is missing from your mirror when a
traveller lands on it, back-fill that single record with `products`, or a batch of them with
`productsBulk`, rather than re-running the whole feed.

## Filtering products out

Filter at ingestion time, not at render time. Two filters matter most:

- **Manual-confirmation products.** If your checkout cannot hold a traveller in a pending state,
  drop products whose confirmation type is not instant.
- **Products you cannot fulfil.** Anything whose booking questions you cannot collect.

## Resolving references

Product records carry opaque references that must be resolved separately, and each resolves through
a different operation:

| Reference | Resolve with |
|---|---|
| `tags[]` (numeric tag ids) | `productsTags` — fetch the whole taxonomy once and cache it |
| `destinations[]` (numeric destination ids) | `destinations` — fetch the whole taxonomy once and cache it |
| `LOC-…` location references | `locationsBulk` — batch them, never one at a time |

`locationsBulk` also accepts the reserved literals `CONTACT_SUPPLIER_LATER` and
`MEET_AT_DEPARTURE_POINT`; treat those as sentinel values, not as lookups.

## Availability schedules

Mirror availability with the same three-way pattern:

1. Initial load with `availabilitySchedulesModifiedSince`, cursored exactly as above.
2. Incremental updates with the stored cursor.
3. Back-fill a single product with `availabilitySchedules`, or a batch with
   `availabilitySchedulesBulk`.

Schedules are a *forecast*. They are what you render on a product page. They are **not** what you
book against — see the booking skill.

## Rate limiting

Limits are per-endpoint, per-PUID, on a rolling 10-second window, and Viator does not publish the
numbers. Read `RateLimit-Limit`, `RateLimit-Remaining` and `RateLimit-Reset` on **successful**
responses and pace the ingestion from those, rather than waiting for the 429. On a 429
(`TOO_MANY_REQUESTS`) or a 503 (`SERVICE_UNAVAILABLE`) honour `Retry-After` and back off; the 503 is
system-wide capacity shedding and is not your fault.

## Content restriction you must honour

Anything inside `viatorUniqueContent`, and any review text, is licensed and carries a published
usage restriction: it must be loaded through an external JavaScript blocked in `robots.txt` and must
not appear in your page source. Store it if you like; do not let a crawler see it.

## Errors

Error bodies are `{code, message, timestamp, trackingId}`. Log `trackingId` — Viator asks for it on
support requests. See `errors/viator-problem-types.yml`.
