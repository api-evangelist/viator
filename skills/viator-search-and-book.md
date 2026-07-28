---
name: Search, check availability and complete a Viator booking
description: >-
  Take a traveller from a search or free-text query through a real-time availability and price
  check, a booking hold, and a confirmed booking — including the timeout and duplicate-suppression
  rules that make this flow safe to retry.
api: openapi/viator-partner-api-v2-openapi.json
generated: '2026-07-28'
method: generated
source: >-
  https://docs.viator.com/partner-api/technical/ (Making a booking, Booking confirmation types,
  Endpoint timeout settings, Booking questions) plus verified operationIds from
  openapi/viator-partner-api-v2-openapi.json
operations:
  - productsSearch
  - searchFreeText
  - products
  - productsBookingQuestions
  - availabilityCheck
  - bookingsHold
  - bookingsBook
  - bookingsStatus
tier: Merchant partner (bookingsHold and bookingsBook are merchant-only)
---

# Search, check availability and complete a Viator booking

`bookingsHold` and `bookingsBook` are **merchant-partner only**. Affiliates with Full-access +
Booking use the cart flow instead — see `viator-cart-booking-and-payment.md`.

## Standing rules

- `Accept: application/json;version=2.0` on every call, `exp-api-key` on every call.
- All development happens on `https://api.sandbox.viator.com/partner`. Production is for live
  bookings only.
- Set a client timeout of **120 seconds**, not the usual 30. Viator brokers to third-party supplier
  systems on the booking call.

## 1. Find a product

Two entry points:

- `productsSearch` — structured filtering by `destination`, `tags`, `flags`, price range, date
  range, rating and duration, with `sorting` and `pagination`.
- `searchFreeText` — a single `searchTerm` across products, destinations and attractions, with
  per-`searchType` pagination.

Both take a `currency`. Set it once and keep it consistent through the whole flow.

## 2. Read the product

Call `products` with the `productCode`. You need three things out of it:

- the `productOptions[]` and their `productOptionCode`s — the traveller is booking an option, not a
  product;
- the `bookingQuestions` that apply, and the valid `languageGuides[]` for the chosen option;
- the `cancellationPolicy`, which you must show before payment.

`productsBookingQuestions` returns the full question catalogue if you want to render the form
generically rather than per product.

## 3. Check real-time availability

Call `availabilityCheck` with `productCode`, `travelDate`, `currency` and `paxMix` (an array of
`{ageBand, numberOfTravelers}`).

This — not the cached availability schedule — is what you may quote and book against. The schedule
is a forecast; `availabilityCheck` is the live truth, and it returns the price you must show.

## 4. Hold

Call `bookingsHold` with `productCode`, `productOptionCode`, `travelDate`, `currency` and `paxMix`.
It returns a Viator `bookingRef` in the form `BR-123456789`. The hold reserves inventory while the
traveller completes checkout.

## 5. Confirm

Call `bookingsBook` with:

- the `bookingRef` from the hold;
- a **`partnerBookingRef` you generate**, unique in your system;
- `bookerInfo` (`firstName` is required), `communication` (`email`, `phone`);
- `bookingQuestionAnswers[]` — per-traveller answers carry a `travelerNum`; some carry a `unit`
  (`kg`, `cm`, `LOCATION_REFERENCE`);
- a `languageGuide` chosen from the options the product actually offers.

Optionally attach `additionalBookingDetails.voucherDetails` so the voucher carries your brand and
support contact rather than Viator's.

## 6. Handle the failure modes — this is the part that matters

A 500 or a timeout on `bookingsBook` **does not mean the booking failed**. Responses can exceed 120
seconds.

The correct recovery, in order:

1. Call `bookingsStatus` with your `partnerBookingRef` (or the `bookingRef`) and look for an
   existing booking.
2. Only if there is none, retry `bookingsBook` — **with the same `partnerBookingRef`**.

`partnerBookingRef` must be unique, and Viator enforces that uniqueness, so a retry carrying the
same value cannot create a duplicate booking. That field is your idempotency key. Never mint a fresh
one on retry: that is exactly how double bookings happen.

## 7. Confirmation type

`bookingStatus` may come back `CONFIRMED`, `PENDING` (manual-confirmation product awaiting the
supplier), `ON_HOLD` or `REJECTED`. If you support manual-confirmation products your checkout,
confirmation page and traveller emails all have to represent "pending" honestly.

To exercise that path in sandbox, send `exp-demo: false` — manual-confirmation products then return
`PENDING` instead of auto-confirming. To move a sandbox booking to `CONFIRMED` or `REJECTED`, email
`apitechsupport@viator.com` with the `bookingRef`; there is no self-service simulator.

## Errors

`400 BAD_REQUEST`, `401 UNAUTHORIZED`, `403 FORBIDDEN` (wrong partner tier for this endpoint),
`429 TOO_MANY_REQUESTS`, `503 SERVICE_UNAVAILABLE`. Bodies are
`{code, message, timestamp, trackingId}`. See `errors/viator-problem-types.yml`.
