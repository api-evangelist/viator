---
name: Book a multi-item Viator cart and tokenise payment
description: >-
  The Full-access + Booking affiliate flow — hold a multi-item cart, tokenise the traveller's card
  against the checkout session so raw card data never touches your servers, then confirm the cart
  with the resulting payment token.
api: openapi/viator-partner-api-v2-openapi.json
generated: '2026-07-28'
method: generated
source: >-
  https://docs.viator.com/partner-api/technical/ (Bookings and Payments sections),
  collections/Viator-Affiliate-Booking-API-v2.postman_collection.json, and
  https://partnerresources.viator.com/travel-commerce/merchant/certification/
operations:
  - availabilityCheck
  - bookingsCartHold
  - paymentsCreateToken
  - bookingsCartBook
  - bookingsStatus
tier: Full-access + Booking Affiliate
---

# Book a multi-item Viator cart and tokenise payment

This is the affiliate transactional flow. Viator remains the **merchant of record**; you pass card
details to Viator in a PCI-compliant way and Viator processes the payment. Passing that
responsibility correctly is one of the things Viator's two-part certification checks before you go
live.

## Standing rules

- `Accept: application/json;version=2.0` and `exp-api-key` on every call.
- Sandbox host for all development: `https://api.sandbox.viator.com/partner`.
- 120-second client timeout on the booking call.

## 1. Price the items

Call `availabilityCheck` once per item you intend to put in the cart, with `productCode`,
`travelDate`, `currency` and `paxMix`. Do not price from the cached availability schedule.

## 2. Hold the cart

Call `bookingsCartHold` with:

- `currency` — one currency for the whole cart;
- a `partnerCartRef` you generate;
- `items[]`, each carrying **your own `partnerBookingRef`**, plus `productCode`,
  `productOptionCode`, `travelDate`, `startTime` and `paxMix`;
- `paymentDataSubmissionMode` — `PARTNER_FORM` when you collect the card on your own form.

The response returns a Viator `cartRef` in the form `CR-…`, a Viator `bookingRef` per item, and the
`paymentDataSubmissionUrl` for the next step.

## 3. Tokenise the card

POST the card payload to the `paymentDataSubmissionUrl` returned by the hold — this is the
`paymentsCreateToken` operation, `/v1/checkoutsessions/{sessionToken}/paymentaccounts`. Send:

- header `x-trip-requestid` — a fresh UUID per request;
- header `x-trip-clientid` — your Viator-issued client id;
- body `paymentAccounts.creditCards[]` with `number`, `cvv`, `expMonth`, `expYear`, `name` and
  `address` (`country`, `postalCode`).

You get back a payment token in the form `STK-…`.

Never log, persist or forward the PAN, and never route this call through anything that is not inside
your PCI scope. If you are building an agent, do not expose this operation as a tool.

## 4. Confirm the cart

Call `bookingsCartBook` with:

- the `cartRef` from step 2;
- `bookerInfo` and `communication`;
- `items[]`, each with the Viator `bookingRef` from the hold, its `bookingQuestionAnswers[]` and its
  `languageGuide`;
- the `paymentToken` from step 3.

## 5. Handle rejection and timeout

A cart book can be rejected for payment reasons, and the rejection vocabulary is separate from the
error envelope. Current values:

`SUSPECTED_FRAUD`, `SOFT_DECLINE`, `HARD_DECLINE`, `THREE_D_SECURE_REQUIRED`, `INTERNAL_ERROR`,
`PROCESSOR_UNAVAILABLE`, `PROCESSOR_ISSUE_WITH_PAYMENT`, `INSUFFICIENT_FUNDS`,
`INVALID_PAYMENT_DETAILS`.

Distinguish soft from hard: a `SOFT_DECLINE` or `PROCESSOR_UNAVAILABLE` is worth a retry with a new
token; a `HARD_DECLINE` or `SUSPECTED_FRAUD` is not. `THREE_D_SECURE_REQUIRED` means the traveller
must complete an authentication step.

On a timeout or HTTP 500, **do not immediately retry**. Call `bookingsStatus` first. If you must
retry, reuse the same `partnerBookingRef` values — Viator enforces their uniqueness, so the retry
cannot create duplicate bookings.

## 6. Certification

Before this integration can go live Viator requires a front-end and a back-end certification form,
emailed to the API onboarding team (`affiliateapi@tripadvisor.com`), and Viator verifies every
requirement itself. Build the pending/rejected states and the cancellation-policy display properly
the first time; they are what certification looks at.

## Test values

Viator's own Full-access + Booking Postman collection ships the standard Visa test PAN
`4111111111111111` for the tokenisation call. Everything published is recorded in
`sandbox/viator-sandbox.yml`.
