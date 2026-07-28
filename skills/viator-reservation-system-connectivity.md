---
name: Implement Viator reservation-system connectivity (supplier side)
description: >-
  The inverted contract — build and expose the endpoints Viator calls into a tour operator's
  reservation system, meet the published SLAs, push inventory and pricing changes to Viator's
  notification endpoints, and validate the whole thing with Viator's contract-testing tool.
api: openapi/viator-reservation-system-api-openapi.json
generated: '2026-07-28'
method: generated
source: >-
  https://docs.viator.com/supplier-api/technical/ (What's new, Implementation approach, Connectivity
  overview, API configurations, SLAs, Circuit breakers, Contract testing) plus verified operationIds
  from openapi/viator-reservation-system-api-openapi.json
operations:
  - tourList
  - availabilityCheck
  - calendar
  - specialOffers
  - reserve
  - booking
  - bookingAmendment
  - bookingCancellation
  - redemption
  - eventNotification
  - specialOffersNotification
tier: Approved reservation system / booking software vendor
---

# Implement Viator reservation-system connectivity (supplier side)

This specification is **inverted**. You do not call these endpoints — you *build* them, and Viator
calls you. That is why `servers[]` is the placeholder
`https://your-reservation-system.example.com`: the implementing host is yours.

## Prerequisites you cannot skip

- Access is restricted to operators registered with Viator and their authorised reservation-system
  providers.
- Integration proceeds only after technical evaluation and formal approval by Viator.
- Development may only commence if Viator-registered operators are **already using** your
  reservation system. This is a demand precondition on a supply-side integration.
- Viator issues your `API key` and each operator's `SupplierId` at the kick-off meeting.

## Authenticate

v2.0 endpoints carry the key in the `X-Api-Key` request header. v1.0 endpoints accept it in the
header too. All v2.0 traffic is JSON — `Content-Type` and `Accept` must be `application/json`. XML
and the legacy SOAP WSDL/XSD interface are deprecated and unavailable to new integrations.

## Endpoints you must implement

Mandatory:

| Operation | Path | Purpose |
|---|---|---|
| `availabilityCheck` | `POST /v2/availability/check` | Real-time capacity through the booking funnel |
| `calendar` | `POST /v2/availability/calendar` | Long-term availability, pricing and offers |
| `reserve` | `POST /v2/reserve` | Hold inventory **and price** during checkout |
| `booking` | `POST /booking` | Create the booking after payment |
| `bookingCancellation` | `POST /booking-cancellation` | Cancel and free inventory |
| `redemption` | `POST /redemption` | Redemption status at cancellation time |

Optional but strongly expected:

| Operation | Path | Purpose |
|---|---|---|
| `tourList` | `POST /tourlist` | Publish your product identifiers so Viator can map inventory |
| `specialOffers` | `POST /v2/product/special-offers` | Special-offer metadata |
| `bookingAmendment` | `POST /booking-amendment` | Apply post-confirmation changes |

Do not build the v1.0 endpoints. `availability`, `batch-availability`, `batch-pricing` and
`availabilitynotification2` are deprecated and superseded by `availabilityCheck`/`reserve`,
`calendar`, `calendar` and `eventNotification` respectively.

## Identify products correctly

`SupplierProductCode` is the only mandatory product identification field — **your** key, not
Viator's, which is why Viator absorbs the mapping cost. Maximum 50 characters, and it must not
contain a pipe (`|`); longer codes fail. Use `SupplierOptionCode` for variants, or the new v2.0
`productOptionId` single identifier, which must be returned in `tourList` and will eventually
replace `SupplierOptionCode` and the `Name/Value` pairs.

Map your price rates into Viator's five: Adult, Child, Youth, Infant, Senior. Concession, Veteran
and Pensioner map to Senior; Student and Teenager to Youth; Toddler and Baby to Infant; Everyone and
Other to Adult. Family pricing is not supported. Car, Boat, Vehicle and Group map to Unit pricing.

Mapped inventory **must** have capacity. A product that will not map is almost always a `tourList`
response problem or a capacity problem, not a Viator problem.

## Push changes instead of waiting to be polled

Two Viator-hosted notification endpoints — Viator's own documentation calls these Notification
Webhooks — let you tell Viator the moment something changes:

- `eventNotification` (`POST /v2/notification/events`) — salability, capacity and pricing changes.
- `specialOffersNotification` (`POST /v2/notification/special-offers`) — new or updated offers.

Both are Beta. Using them is what prevents overbookings and surfaces last-minute availability.

## Meet the SLA, or get circuit-broken

| Capability | P90 latency | Max monthly error rate |
|---|---|---|
| Availability Check | < 1s | < 1.5% |
| Reserve | < 1s | < 1.5% |
| Booking (incl. amendment, redemption, cancellation) | < 5s | < 0.75% |
| Calendar | < 5s | < 1% |
| Tourlist | < 10s | < 0.5% |
| Event / Special Offers Notification | < 1s | < 0.5% |

Uptime target is 99.8% for critical synchronous capabilities, 99.5% for asynchronous ones.
Throughput scales with a product multiplier `X = ceil(connected products / 10,000)`, minimum 1 — for
example Availability Check is `X × 10` RPS sustainable, `X × 20` peak.

If a capability crosses an unacceptable threshold Viator **opens a circuit breaker** and stops
sending requests to it entirely; the failure-rate threshold is 70%. Viator can cut your operators'
distribution off unilaterally, so treat these numbers as the contract they are.

## Release inventory you are holding

`reserve` supersedes the v1.0 `AvailabilityHold`. Hold inventory long enough for checkout to
complete, and release it if the booking is not confirmed within a predefined duration. Holding
forever is as much a failure as not holding at all.

## Validate before you certify

Viator ships a self-service contract-testing tool as a local Docker web app:

```bash
export API_KEY=<your-api-key>
export SUPPLIER_ID=<your-supplierId>
docker pull public.ecr.aws/viator/cica:latest
docker run -it --rm --name viator-contract-testing -p 5173:5173 -p 8989:8989 \
  -e API_KEY=$API_KEY -e SUPPLIER_ID=$SUPPLIER_ID \
  --add-host=host.docker.internal:host-gateway \
  public.ecr.aws/viator/cica:latest
```

Open `http://localhost:5173`. Your endpoints do not need to be publicly exposed — point the tool at
`http://host.docker.internal:<port>/your-api-path`. Inputs resolve Test Case Input > Test Product
Option Config > Global Defaults. v1 endpoints are not supported by the tool.

Support: `supplierAPI@viator.com`.
