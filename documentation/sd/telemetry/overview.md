# Telemetry overview (HIGH)

Every backend service in the Pear Store has the **option** to publish
OpenTelemetry-shaped signals through a small in-process shim
(`shared/otel.py`). The shim is dependency-free: it does not import
the real `opentelemetry` package. It exists so the synthetic
demo can produce realistic telemetry shapes without taking on the
weight of the full SDK.

> **Don't copy this pattern into a real system.** A production
> deployment would use the real OpenTelemetry SDK with an OTLP
> exporter pointing at a collector. The shim's job is to make the
> *observability layer* of the synthetic data look familiar enough
> for a documentation-gap agent to reason about.

## Signals

Two OTel signal types are exercised:

### Tracing

Every instrumented service registers a Flask `before_request`/
`after_request` pair that emits one **root span** per HTTP request:

| Field             | Meaning                                                           |
| ----------------- | ----------------------------------------------------------------- |
| `name`            | `"<service> <method> <route>"` (e.g. `"order POST /checkout"`)    |
| `trace_id`        | hex; lifted from the inbound `traceparent` header if present      |
| `span_id`         | hex                                                               |
| `parent_id`       | the span id from `traceparent`, or `null` for a request that originates a trace |
| `start_ns` / `end_ns` / `duration_ms` | wall-clock window of the request                      |
| `status`          | `"OK"` if HTTP < 400, `"ERROR"` otherwise                         |
| `attributes`      | `http.method`, `http.route`, `http.status_code`, plus a derived `status` of `"success"` / `"failure"` |

Services may also emit **child spans** with `child_span(tracer, name, attrs)`.
Children get the request's trace id and parent id automatically. Today
only `order` and `payment` emit child spans — see `order.md` and
`payment.md`.

### Metrics

Every instrumented service publishes two request-level signals:

| Metric                                | Kind        | Attributes                                                    |
| ------------------------------------- | ----------- | ------------------------------------------------------------- |
| `<service>_requests_total`            | counter     | `http.method`, `http.route`, `http.status_code`, `status`     |
| `<service>_request_duration_ms`       | histogram   | same as above                                                 |

`status` is `"success"` if `http.status_code < 400`, else `"failure"`.
That is how success vs failure rate is computed downstream.

Services may also publish **business metrics**. Those are listed in
the per-service docs — for example `order_checkout_outcomes_total`,
`payment_amount_cents`, or `fulfillment_licenses_issued_total`.

## Trace propagation

Outgoing HTTP from an instrumented service should set a
`traceparent` header so the downstream span stitches into the
same trace. The helper:

```python
from shared.otel import traceparent_header
requests.get(url, headers=traceparent_header(), timeout=5)
```

Today `order` is the only originator that propagates everywhere it
calls. Whether the downstream service honours the header depends on
whether *it* is instrumented.

## Output format

All signals are written to `synthetic-data/telemetry/<service>/` as
append-only JSON Lines (`traces.jsonl`, `metrics.jsonl`). One record
per line, no batching. The directory is created lazily the first
time each service emits.

## Coverage tiers

The fleet is split into four tiers on purpose:

| Tier    | Services            | What they emit                                           |
| ------- | ------------------- | -------------------------------------------------------- |
| HIGH    | order, payment      | root spans, child spans for downstreams, full metrics    |
| MEDIUM  | fulfillment, pearcare-claim | root spans + metrics; business counters; **no child spans** |
| LOW     | catalog             | metrics only — no traces                                 |
| NONE    | account, cart, review, search, pearcare-plan | nothing                                                  |

The tiering is what makes it possible to run the demo against a
gap-detection agent: the agent should notice that `account` /
`cart` / `review` / `search` / `pearcare-plan` produce no telemetry,
that catalog has no traces, and that the medium-tier services are
opaque past the request boundary.

## Ownership boundaries

Services responsible for emitting root spans, child spans, metrics, and business counters include:

* The Catalog service produces event logs, system metrics, and request traces [S2](../services/catalog.md). Event logs from the catalog service can be used for monitoring interactions between the review service and the catalog service, as well as interactions with other services via the catalog API [S1](README.md) [S5](fulfillment.md).
* The Account service generates audit records, performance indicators, and distributed request data for monitoring and troubleshooting purposes. Specifically:
	+ Audit records are generated when orders are persisted in the `failed` state for a certain period [S1](README.md) [S2](../services/catalog.md). These records imply data retention for auditing purposes.
	+ Performance indicators may include metrics related to user cart management, such as the number of carts created or updated by the catalog service [S4](../services/account.md).
	+ Distributed request data may be generated when the account service interacts with other services, including the catalog service, which is responsible for managing user carts and retrieving product information [S3](catalog.md) [S5](fulfillment.md).
* The Fulfillment service emits the standard Flask request-level signals plus a business counter for licenses issued. There is no per-app instrumentation inside `/fulfill` [S5](fulfillment.md). Only the root span is emitted.
* The Review service emits event logs related to interactions with the catalog service via the catalog API [S1](README.md).

The following services are responsible for emitting metrics and business counters:

* The Catalog service produces system metrics, including CPU usage or memory consumption [S2](../services/catalog.md) (further detail TBD).
* The Fulfillment service emits a business counter for licenses issued [S5](fulfillment.md).
* The Account service generates performance indicators related to user cart management [S4](../services/account.md).

The following services are responsible for emitting request traces:

* The Catalog service produces request traces, which can help identify bottlenecks in cart management processes [S2](../services/catalog.md) (further detail TBD).
* The Fulfillment service does not emit child spans or per-app instrumentation inside `/fulfill` [S5](fulfillment.md).

## Why this shape

<!-- SME-PLACEHOLDER:q-sd-35258849b6 START -->
> ⏳ **Waiting for SME** — *Topic:* Why this shape
>
> *Question:* What are the design decisions behind splitting the fleet into four tiers (HIGH, MEDIUM, LOW, NONE)?
> *Best guess (low-confidence):* The design rationales for categorizing vessels into high-priority, medium-priority, low-priority, and no-priority groups are not explicitly mentioned in the provided sources. However, the sources do discuss the design of a standalone subsystem for PearCare management, which was driven by the need to modularize and decouple services [S1]. The key factors driving this design decision were:

* Reusing fulfillment logic from PearCare's claim service [S2], [S5]
* Implementing a modular, service-oriented architecture where each service has a specific responsibility and can be developed and maintained independently
* Pushing ratings updates instead of pulling them [S2]

The standalone subsystem for PearCare management was designed to handle the claim lifecycle, including filing, triage, approval/denial, and fulfillment. However, there is no mention of categorizing vessels into priority groups in the provided sources.

Note: The sources do not provide information on categorizing vessels into high-priority, medium-priority, low-priority, and no-priority groups.
> *Asked:* on 2026-06-27 · *Status:* pending · *Question ID:* `q-sd-35258849b6`
<!-- SME-PLACEHOLDER:q-sd-35258849b6 END -->

## What is not in this system

Observability aspects not covered by the telemetry system include:

* Disaster recovery procedures [S4](../database/payment-db.md).
* Data retention policies for orders in the `failed` state [S6](../runbooks/pearcare-fraud.md) (further detail TBD).
* Per-app instrumentation inside `/fulfill` for the fulfillment service [S7](fulfillment.md).
* Counter for receipt reads vs library reads for the fulfillment service [S7](fulfillment.md).
* Aggregation of metrics by app_id for the fulfillment service, which requires going to the database instead of relying on metrics [S7](fulfillment.md).

Note that these aspects are mentioned as unresolved issues or areas that need SME input in the telemetry documentation [S2](README.md).
