# Business case: catalog discovery

## Goal

Get a user from "I opened the store" to "I opened an app detail page"
in as few decisions as possible. Discovery is the top of the purchase
funnel; if it leaks, every downstream conversion metric leaks with it.

## Audience

Two persona shapes:

* **Browsers** — no specific app in mind. They scroll Top Charts and
  category strips. Conversion is driven by *what we surface*.
* **Hunters** — they typed a name into the search bar. Conversion is
  driven by *whether we found it*. Latency matters more than ranking.

## How the system supports it

* `GET /top-charts` on catalog-svc gives the home page a stable,
  popularity-ordered slice. We don't personalize; ordering is purely
  `rating_count`. This is intentional for now — until we have logged-in
  behavior, personalization would be guessing.
* Categories are the secondary navigation. They live on catalog-svc to
  keep the home page render to one upstream call.
* Search is a separate service so we can iterate on ranking without
  redeploying catalog. The home page never calls search.
* We push the rating snapshot from review-svc back into catalog so
  ranking does not require a fan-out per result.

## Business levers

* **Editorial overrides** (not yet implemented): pin certain apps to the
  top of charts during a launch window. Would belong in catalog-svc as
  a `POST /charts/pin` admin endpoint.
* **Category strips on the home page**: a future home page could pull
  the top three apps per category. Today we only show the flat top
  chart and a category navigation row.
* **Search synonyms**: today's tokenizer is exact-match-only. A
  synonyms file would meaningfully improve hunter conversion.

## Measurable outcomes

* Time-to-first-detail-page-view (T1) — the dominant funnel metric.
  Targets to keep the system under pressure to stay quick.
* Search-to-detail conversion (search results clicked / searches).
* Top-chart click-through rate.

## Stakeholders

Key stakeholders involved in implementing catalog discovery include:

* Decision-makers:
	+ Catalog Service team, responsible for managing user carts and retrieving product information [S2](../../sd/telemetry/catalog.md)
	+ Account Service team, generating audit records, performance indicators, and distributed request data for monitoring and troubleshooting purposes [S4](../../sd/telemetry/overview.md)
	+ Review Service team, emitting event logs related to interactions with the catalog service via the catalog API [S1](../../sd/telemetry/README.md)
* Funders:
	+ Not explicitly mentioned in the provided sources
* Other stakeholders:
	+ Search Service team, using system metrics from the catalog service to monitor its execution time, throughput, and latency [S2](../../sd/telemetry/catalog.md)
	+ Cart Service team, responsible for managing user carts and storing information about items a user has added to their cart [S9]

Note that the specific roles and responsibilities of these stakeholders may not be fully defined in the provided sources.

## Success metrics

<!-- SME-PLACEHOLDER:q-bp-aeeee1bb6a START -->
> ⏳ **Waiting for SME** — *Topic:* Success metrics
>
> *Question:* What additional success metrics will be used to measure the effectiveness of catalog discovery?
> *Best guess (low-confidence):* Additional success metrics that will be used to measure the effectiveness of catalog discovery include:

* Error logs related to interactions with the catalog service via the catalog API [S1], which can provide insights into the performance of the catalog service, such as CPU usage or memory consumption [S2].
* System metrics from the catalog service, including CPU usage and memory consumption [S3] (further detail TBD).
* Request traces from the catalog service, which can help identify bottlenecks in cart management processes [S5].

These metrics can be used to monitor interactions between the review service and the catalog service, as well as interactions with other services via the catalog API [S4]. The search monitoring, analytics, or logging service can use this information to monitor its execution time, throughput, and latency by analyzing error logs, system metrics, and request/response traces from the catalog service.

Typical query shapes for accessing apps in the catalog database include retrieving app information by ID, such as `SELECT * FROM catalog WHERE app_id = ?` [S3], which can provide insights into the performance of the catalog service. The most frequently accessed fields are likely to be `app_id`, `price_cents`, and other attributes related to product information [S2].

The search monitoring, analytics, or logging service can use this information to monitor its execution time, throughput, and latency by analyzing error logs, system metrics, and request/response traces from the catalog service.

Note that the specific data stored in the cart database for each user is unclear (further detail TBD) [S4], but the Cart service stores information about items a user has added to their cart, including `user_id`, `items`, and other relevant data [S9].
> *Asked:* on 2026-06-27 · *Status:* pending · *Question ID:* `q-bp-aeeee1bb6a`
<!-- SME-PLACEHOLDER:q-bp-aeeee1bb6a END -->

## Risks

Potential Risks and Mitigations for Implementing Catalog Discovery:

**Technical Considerations**

Implementing catalog discovery may introduce technical risks, including:

* Data transmission delays: The telemetry system transmits collected data to a central location for analysis, which may involve interactions between services. This could lead to delays in data processing and analysis [S2](../../sd/telemetry/README.md).
* Data loss or corruption: Event logs, system metrics, and request traces are generated by various services, including the catalog service. If these data streams are not properly managed, there is a risk of data loss or corruption [S1](../../sd/services/catalog.md).

Mitigations:

* Implement robust data transmission protocols to minimize delays.
* Regularly monitor and maintain data streams to prevent loss or corruption.

**Business Considerations**

Implementing catalog discovery may also introduce business risks, including:

* Data retention: Audit records are generated when orders are persisted in the `failed` state for a certain period [S6](../../sd/telemetry/overview.md). This implies that data is retained for auditing purposes.
* Performance indicators: Performance indicators may include metrics related to user cart management, such as the number of carts created or updated by the catalog service [S4](../../sd/telemetry/search.md).

Mitigations:

* Regularly review and update data retention policies to ensure compliance with business requirements.
* Monitor performance indicators to identify areas for improvement.

**Operational Considerations**

Implementing catalog discovery may also introduce operational risks, including:

* System metrics: System metrics from the catalog service provide insights into its performance, such as CPU usage or memory consumption [S2](../../sd/telemetry/README.md). These metrics can be used by the Search service to monitor its execution time, throughput, and latency.
* Request traces: Request traces from the catalog service help identify bottlenecks in cart management processes [S5](../../sd/telemetry/catalog.md).

Mitigations:

* Regularly review system metrics to identify areas for improvement.
* Use request traces to optimize cart management processes.

Note that the specific data stored in the cart database for each user is unclear (further detail TBD) [S4](../../sd/telemetry/search.md).

## Open Questions

Outstanding unanswered queries or unresolved matters concerning catalogue identification that necessitate subject matter expert (SME) involvement include:

* Clarifying the consistency model [S1](../../sd/database/catalog-db.md), [S3](../../sd/database/catalog-db.md)
* Determining the specific data stored in the cart database [S1](../../sd/database/catalog-db.md), [S3](../../sd/database/catalog-db.md)
* Identifying the data stored in the account database [S1](../../sd/database/catalog-db.md), [S3](../../sd/database/catalog-db.md)
* Refining triage rules to better categorize and prioritize customer issues based on keywords [S2](../../sd/services/search.md), [S5](../../sd/services/review.md), [S9](../../sd/runbooks/incident-response.md)
* Improving payment processing by handling declined payments more efficiently [S5](../../sd/services/review.md), [S9](../../sd/runbooks/incident-response.md)
* Enhancing order fulfillment by automating refunds upon fulfillment failure [S2](../../sd/services/search.md), [S5](../../sd/services/review.md), [S9](../../sd/runbooks/incident-response.md)

Additionally, there are areas where further detail is TBD regarding:

* Indexing strategies for the databases [S4](../../sd/services/catalog.md)
* Backup strategies for the databases [S5](../../sd/services/review.md)
* Index and backup strategies [S5](../../sd/services/review.md)
* The exact duration of data retention for orders in the `failed` state [S6](../../sd/telemetry/overview.md), [S9](../../sd/runbooks/incident-response.md) (further detail TBD)
