---
name: Amend, cancel and reconcile Viator bookings
description: >-
  Post-booking operations — quote and apply an amendment, quote and take a cancellation, and consume
  the booking event feed with its acknowledgement cursor so supplier-initiated changes are never
  silently dropped.
api: openapi/viator-partner-api-v2-openapi.json
generated: '2026-07-28'
method: generated
source: >-
  https://docs.viator.com/partner-api/technical/ (Cancellation API workflow, Checking booking
  status, Updates) plus verified operationIds and the BookingEvent schema from
  openapi/viator-partner-api-v2-openapi.json
operations:
  - amendmentCheck
  - amendmentQuote
  - amendmentAmend
  - bookingsCancelReasons
  - bookingsCancelQuote
  - bookingsCancel
  - bookingsStatus
  - bookingsModifiedSince
  - bookingsModifiedSinceAcknowledge
tier: Full-access + Booking Affiliate or Merchant partner
---

# Amend, cancel and reconcile Viator bookings

Every mutating step here is **quote first, commit second**. Never call the commit operation without
showing the traveller what the quote returned.

## Amend

1. `amendmentCheck` with the `booking-reference` (`BR-…`) — tells you whether this booking can be
   amended at all.
2. `amendmentQuote` with the `bookingRef` and exactly one kind of change:
   - `updatePaxMix` — `removePaxMix[].travelerNum` and/or `addPaxMix[]` with `ageBand`,
     `travelerNum` and that traveller's `bookingQuestions`;
   - `bookingDetails` — `productOptionCode`, `travelDate`, `startTime`;
   - `perTravelerQuestions` — per-traveller answer corrections;
   - `perBookingQuestions` — pickup point, transfer arrival/departure details.
   Set `voucherFormat` (e.g. `PDF`). The response returns a `quoteRef` in the form `QR-<uuid>` and
   the price impact.
3. `amendmentAmend` with the `quote-reference` to commit.

Permissible amendments on the supply side are the lead traveller's name, the traveller mix, the
travel date, hotel details and the tour option. Anything else is a cancel-and-rebook.

## Cancel

1. `bookingsCancelReasons` — fetch the reason-code list. Optionally filter with `type=SUPPLIER`.
   Cache it; it is reference data.
2. `bookingsCancelQuote` with the `booking-reference` — returns the refund amount and percentage
   **before** you commit. Show this to the traveller.
3. `bookingsCancel` with the `booking-reference` and a `reasonCode` taken verbatim from step 1
   (e.g. `Customer_Service.Chose_a_different_cheaper_tour`).

Cancellation terms come from the product's `cancellationPolicy`: `STANDARD`, `CUSTOM`, or all-sales-
final with a 100% penalty. Post-travel cancellations and partial refunds are handled by the same
quote-then-commit pair.

## Reconcile — the booking event feed

`bookingsModifiedSince` is a cursored feed of things that happened **to** your bookings. By default
it excludes events you initiated through the API, so it is a change feed of supplier and customer
action, not an echo of your own writes.

Each `BookingEvent` carries `transactionRef` (`PBE-<uuid>`), `eventType`, `bookingRef`,
`partnerBookingRef`, `lastUpdated`, `acknowledgeBy` and `bookedItem`.

| `eventType` | Meaning | Acknowledge? |
|---|---|---|
| `CONFIRMATION` | Booking confirmed | No |
| `REJECTION` | Supplier rejected the booking | No |
| `AMENDMENT` | Booking amended outside the API | No |
| `CANCELLATION` | **Supplier** cancelled | **Yes, before `acknowledgeBy`** |
| `CUSTOMER_CANCELLATION` | Customer cancelled outside the API | No |

Loop:

1. `bookingsModifiedSince` with `count` and the stored `cursor` (first run: neither `cursor` nor
   `modified-since`).
2. Apply each event to your local booking record, keyed on `bookingRef` or `partnerBookingRef`.
3. `bookingsModifiedSinceAcknowledge` with the `transactionRefs` you have durably processed.
4. Store the new cursor.

**Acknowledge deliberately, not reflexively.** Acknowledging a supplier `CANCELLATION` before its
`acknowledgeBy` timestamp takes ownership of the customer-communication obligation for that
cancellation. If you are not actually going to email the traveller, do not acknowledge it. This is
the one call in the Viator surface whose side effect is contractual rather than technical.

For a supplier cancellation the refund is always 100%; `cancellationReasonCode` resolves to natural
language through `bookingsCancelReasons`.

## Exit path

This feed plus `bookingsStatus` is also how you get your data out. `bookingsModifiedSince` from the
beginning of the cursor, drained to completion with the acknowledgement cursor, is a documented,
complete export of your own booking records. The catalogue is not exportable — it is licensed
content you ingest, not an asset you own.
