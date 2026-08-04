# Viator (viator)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Viator is a Tripadvisor company and the largest online marketplace for tours, activities and travel experiences, headquartered in the United States and listing more than 300,000 bookable products across roughly 2,500 destinations. It sits on the demand side of the travel distribution chain as an aggregator and reseller of third-party operator inventory, and on the supply side as the channel counterparty that tour operators' reservation systems connect into. Its API posture is unusually open for travel: the full Viator Partner API v2 OpenAPI, the legacy v1 affiliate and merchant specifications, the Viator Reservation System (supplier) API and four Postman collections are all published without a login at docs.viator.com, and Basic Access affiliate keys are issued self-serve at no cost on account creation. Everything beyond that is gated — Full Access, Full Access plus Booking, Merchant and supplier connectivity all require qualification by Viator and, for transactional integrations, passing a two-part front-end and back-end certification. No open travel standard is referenced anywhere in the specifications: the contract is entirely Viator-proprietary, product identifiers are Viator-internal, and partners are contractually required to prevent search engines indexing Viator reviews and unique content.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/viator/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/viator/refs/heads/main/apis.yml)

## Tags

- Travel
- United States
- Tours and Activities
- Experiences
- OTA
- Booking
- Distribution
- Marketplace
- Affiliate
- Hospitality

## Timestamps

- **Created:** 2026-07-28
- **Modified:** 2026-07-28

## APIs

### Viator Partner Products API

Product content and catalogue ingestion for the Viator Partner API v2 — full product detail, bulk retrieval, incremental modified-since ingestion, the product tag taxonomy, booking questions, product search and product-to-product recommendations.

- **Human URL:** [https://docs.viator.com/partner-api/technical/](https://docs.viator.com/partner-api/technical/)
- **Base URL:** `https://api.viator.com/partner`

#### Tags

- Products
- Catalog
- Content
- Search

#### Properties

- [OpenAPI](openapi/viator-partner-api-v2-openapi.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://docs.viator.com/partner-api/technical/)
- [API Reference](https://docs.viator.com/partner-api/technical/#tag/Products)
- [Postman Collection](collections/Viator-Merchant-API-v2.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)

### Viator Partner Availability API

Real-time availability and price checking plus availability-schedule retrieval for a single product, in bulk, or incrementally by modification date, so partners can hold a local mirror of bookable capacity and pricing.

- **Human URL:** [https://docs.viator.com/partner-api/technical/](https://docs.viator.com/partner-api/technical/)
- **Base URL:** `https://api.viator.com/partner`

#### Tags

- Availability
- Pricing
- Schedules

#### Properties

- [OpenAPI](openapi/viator-partner-api-v2-openapi.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://docs.viator.com/partner-api/technical/)
- [API Reference](https://docs.viator.com/partner-api/technical/#tag/Availability)

### Viator Partner Bookings API

Transactional booking surface — cart and single-item hold and book, booking status, cancel reasons, cancellation quote and cancellation, amendment check, quote and amend, and the modified-since booking feed with an acknowledgement cursor for pulling a partner's own booking records.

- **Human URL:** [https://docs.viator.com/partner-api/technical/](https://docs.viator.com/partner-api/technical/)
- **Base URL:** `https://api.viator.com/partner`

#### Tags

- Bookings
- Orders
- Cancellation
- Amendments

#### Properties

- [OpenAPI](openapi/viator-partner-api-v2-openapi.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://docs.viator.com/partner-api/technical/)
- [API Reference](https://docs.viator.com/partner-api/technical/#tag/Bookings)
- [Postman Collection](collections/Viator-Affiliate-Booking-API-v2.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)

### Viator Partner Payments API

Checkout-session payment account endpoint used by Full Access plus Booking affiliate partners to pass traveller payment details to Viator in a PCI-compliant way when Viator remains the merchant of record.

- **Human URL:** [https://docs.viator.com/partner-api/technical/](https://docs.viator.com/partner-api/technical/)
- **Base URL:** `https://api.viator.com/partner`

#### Tags

- Payments
- Checkout

#### Properties

- [OpenAPI](openapi/viator-partner-api-v2-openapi.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [API Reference](https://docs.viator.com/partner-api/technical/#tag/Payments)

### Viator Partner Attractions API

Attraction search and attraction detail endpoints, letting partners build attraction landing pages and tie Viator's bookable products back to the places they visit.

- **Human URL:** [https://docs.viator.com/partner-api/technical/](https://docs.viator.com/partner-api/technical/)
- **Base URL:** `https://api.viator.com/partner`

#### Tags

- Attractions
- Places

#### Properties

- [OpenAPI](openapi/viator-partner-api-v2-openapi.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [API Reference](https://docs.viator.com/partner-api/technical/#tag/Attractions)

### Viator Partner Auxiliary API

Supporting reference and content services for the Partner API v2 — free-text search across products, destinations and attractions, bulk location resolution, exchange rates, product reviews, supplier product-code lookup and the destination taxonomy.

- **Human URL:** [https://docs.viator.com/partner-api/technical/](https://docs.viator.com/partner-api/technical/)
- **Base URL:** `https://api.viator.com/partner`

#### Tags

- Search
- Locations
- Reviews
- Exchange Rates
- Destinations

#### Properties

- [OpenAPI](openapi/viator-partner-api-v2-openapi.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [API Reference](https://docs.viator.com/partner-api/technical/#tag/Auxiliary)

### Viator Reservation System API

The supplier-side connectivity contract, formerly the Viator Supplier API. This specification is inverted — it defines the endpoints a tour operator's reservation system or booking software must itself implement and expose for Viator to call, covering tour list mapping, real-time and batch availability, batch pricing, reserve, booking, amendment, cancellation and redemption, alongside the Viator-hosted event and special-offer notification endpoints. It carries published uptime, latency and error-rate SLAs and Viator-operated circuit breakers.

- **Human URL:** [https://docs.viator.com/supplier-api/technical/](https://docs.viator.com/supplier-api/technical/)
- **Base URL:** not applicable — the implementing host is the supplier's own reservation system

#### Tags

- Supplier
- Connectivity
- Channel Manager
- Inventory
- Reservation System

#### Properties

- [OpenAPI](openapi/viator-reservation-system-api-openapi.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://docs.viator.com/supplier-api/technical/)
- [Documentation](https://docs.viator.com/supplier-api/technical/api-overview/)
- [Documentation](https://docs.viator.com/supplier-api/technical/connectivity-overview/index.html)
- [Documentation](https://docs.viator.com/supplier-api/technical/sapi-manual-latest/)

### Viator Merchant API v1

The legacy v1 merchant-partner specification still published by Viator, exposing taxonomy, product, photo, review, availability, pricing-matrix, booking, voucher and cancellation services under viatorapi.viator.com/service. Superseded by the Partner API v2 but retained in the public documentation.

- **Human URL:** [https://docs.viator.com/partner-api/merchant/technical/](https://docs.viator.com/partner-api/merchant/technical/)
- **Base URL:** `https://viatorapi.viator.com/service`

#### Tags

- Merchant
- Legacy
- Bookings
- Products

#### Properties

- [OpenAPI](openapi/viator-merchant-api-v1-openapi.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://docs.viator.com/partner-api/merchant/technical/)

### Viator Affiliate API v1

The legacy v1 affiliate-partner specification, a non-transactional subset covering utility services, destination and category taxonomy, product and attraction search, product detail, reviews and photos under viatorapi.viator.com/service. Superseded by the Partner API v2.

- **Human URL:** [https://docs.viator.com/partner-api/affiliate/technical/](https://docs.viator.com/partner-api/affiliate/technical/)
- **Base URL:** `https://viatorapi.viator.com/service`

#### Tags

- Affiliate
- Legacy
- Products
- Attractions

#### Properties

- [OpenAPI](openapi/viator-affiliate-api-v1-openapi.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://docs.viator.com/partner-api/affiliate/technical/)

## Common Properties

- [Website](https://www.viator.com/)
- [Documentation](https://docs.viator.com/partner-api/technical/)
- [Portal](https://partnerresources.viator.com/)
- [Support](https://partnerhelp.viator.com/en)
- [Blog](https://partnerresources.viator.com/blog/)
- [LinkedIn](https://www.linkedin.com/company/viator)
- [Postman Collection](collections/Viator-Basic-Access-Affiliate-API-v2.postman_collection.json)
- [Postman Collection](collections/Viator-Affiliate-API-v2.postman_collection.json)
- [Postman Collection](collections/Viator-Affiliate-Booking-API-v2.postman_collection.json)
- [Postman Collection](collections/Viator-Merchant-API-v2.postman_collection.json)

## Switching Cost

Recorded in full in [`review.yml`](review.yml).

| Dimension | Finding |
|---|---|
| Interface shape | `proprietary-documented` — four public OpenAPI documents, zero references to OpenTravel/OTA, HTNG, NDC, IATA or GDS in any of them |
| Second source | `alternatives-with-migration` — GetYourGuide, Klook, Tiqets, Musement, Civitatis all sell comparable supply, none share a schema |
| Exit path | `bulk-export-documented` — `GET /bookings/modified-since` plus `POST /bookings/modified-since/acknowledge` export a partner's own bookings; catalogue content is licensed, not portable |
| Identifier portability | Viator-proprietary product codes (`5010SYDNEY`, `46334P42`), Viator destination and attraction IDs, Viator `bookingRef`; the supplier side inverts this and uses the operator's own `SupplierProductCode` |
| Contractual lock-in | Published: the "Protecting unique content" clause forbids letting search engines index Viator reviews; merchant certification is mandatory; supplier SLAs and unilateral circuit breakers. Not published: term, exclusivity, termination, commission rates |
| Access gate | `self-serve` at Basic Access ("By creating an affiliate account, you'll immediately get Basic Access to our API"), application-approval plus certification for every tier above |
| Distribution model | `aggregator-reseller` — reseller downstream, channel-manager counterparty upstream |
| NDC posture | Not applicable — no air content |

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
