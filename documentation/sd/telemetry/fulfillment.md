# fulfillment — telemetry (MEDIUM)

`fulfillment` emits the standard Flask request-level signals plus a
business counter for licenses issued. There is no per-app
instrumentation inside `/fulfill`.

## Spans

Only the root span is emitted. `name` follows
`fulfillment <method> <route>`. Common ones:

- `fulfillment POST /fulfill`
- `fulfillment GET /receipts/<order_id>`
- `fulfillment GET /users/<user_id>/library`

The service honours inbound `traceparent`, so when `order` calls
`/fulfill` the root span attaches to the checkout trace as a sibling
of the payment span.

## Metrics

| Name                                       | Kind      | Attributes                                                |
| ------------------------------------------ | --------- | --------------------------------------------------------- |
| `fulfillment_requests_total`               | counter   | `http.method`, `http.route`, `http.status_code`, `status` |
| `fulfillment_request_duration_ms`          | histogram | same                                                      |
| `fulfillment_licenses_issued_total`        | counter   | `order_id`. Increments by `len(app_ids)` per `/fulfill`.  |

## Trace propagation

Inbound: yes (root span attaches to caller's trace).
Outbound: none — fulfillment talks to the in-process license / iOS /
macOS hooks but those calls are not HTTP and not spanned today.

## Known gaps

- **No per-app spans inside `/fulfill`.** The root span tells you
  fulfillment took 80 ms, but it does not tell you whether the time
  went to `license_hook`, the iOS distribution hook, or the macOS
  distribution hook. If one of those hooks regresses you cannot tell
  from telemetry which one.
- **`fulfillment_licenses_issued_total` has no `app_id` attribute.**
  Aggregating by app requires going to the database, not the metrics.
- **No counter for receipt reads vs library reads.** Both are GETs
  and end up under `fulfillment_requests_total`, sliced only by
  `http.route`.

## Overview

The primary responsibilities of the fulfillment service in terms of telemetry and business counters include:

Managing entitlements and replacements, as it reuses its logic from PearCare's claim service when a claim resolves to a replacement [S4](../runbooks/pearcare-fraud.md), [S5](../database/cart-db.md). This suggests that the fulfillment service plays a key role in managing entitlements and replacements.

Processing replacement orders and sending entitlements in response to POST requests from the CL service [S3](../services/fulfillment.md).

Generating event logs related to interactions with the review service via the catalog API, as well as request traces related to cart management [S6](../services/fulfillment.md).

## Endpoints

The fulfillment service handles the following HTTP methods and routes:

* POST requests from the CL service for replacement resolutions, which trigger the processing of replacement orders and the sending of entitlements in response [S6](../database/pearcare-claim-db.md) (domain=sd, source=sd/database/pearcare-claim-db.md).
* The specific route for this request is not explicitly stated, but it involves sending an order ID for replacement to the Fulfillment Service (FUL) [S7](../architecture/data-flow-purchase.md) (domain=sd, source=sd/architecture/data-flow-purchase.md).

The fulfillment service mints licenses and per-platform download URLs [S3](../database/payment-db.md) (domain=sd, source=sd/database/payment-db.md), and it reuses its logic from PearCare's claim service when a claim resolves to a replacement [S4](../database/cart-db.md) (domain=sd, source=sd/database/cart-db.md). This suggests that the fulfillment service plays a key role in managing entitlements and replacements.

The specific HTTP methods and routes handled by the fulfillment service for minting licenses and per-platform download URLs are not explicitly stated. Further detail is TBD [further detail TBD].

## Details

The fulfillment service emits business counters for licenses issued by incrementing a counter for each license minted. This design choice implies that the fulfillment service is responsible for tracking and reporting on the number of licenses issued, which can be used to monitor and analyze the performance of the system.

When a user initiates a checkout, the order service retrieves the cart contents from the cart service [S3](../pearcare/overview.md). The fulfillment service then mints licenses and per-platform download URLs [S2](../architecture/overview.md), incrementing the business counter for each license issued. This process is repeated for each replacement entitlement processed by the fulfillment service [S4](../services/order.md).

The implications of this design choice are that the fulfillment service must maintain accurate and up-to-date information about the number of licenses issued, which can be used to inform business decisions and optimize system performance. However, further detail regarding the specific data stored in the account database for each user is TBD [S7].
