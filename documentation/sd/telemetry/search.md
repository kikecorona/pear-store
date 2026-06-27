# search — telemetry

Serach service publishes:

- `search_queries_total` — counter, taged with `query_kind`.
- `search_query_duration_ms` — historgram.
- `search_zero_result_total` — counter for empty results.

i think the spans are also wired up but i havent confirmed. we
had a meeting about this last quater.

## Endpoints

The search monitoring, analytics, or logging service exposes the following HTTP API endpoints:

* Error logs related to interactions with the catalog service via the catalog API [S1](fulfillment.md)
* System metrics from the catalog service, which can provide insights into the performance of the catalog service, such as CPU usage or memory consumption [S3](catalog.md) (further detail TBD)
* Request traces from the catalog service, which can help identify bottlenecks in cart management processes [S5](catalog.md)

The search monitoring, analytics, or logging service also uses event logs from the catalog service to monitor interactions between the review service and the catalog service.

Typical query shapes for accessing apps in the catalog database include retrieving app information by ID, such as `SELECT * FROM catalog WHERE app_id = ?` [S3](catalog-db.md). The most frequently accessed fields are likely to be `app_id`, `price_cents`, and other attributes related to product information [S2](../architecture/overview.md).

The search monitoring, analytics, or logging service can use this information to monitor its execution time, throughput, and latency by analyzing error logs, system metrics, and request/response traces from the catalog service.

Note that the specific data stored in the cart database for each user is unclear (further detail TBD) [S4](catalog.md), but the Cart service stores information about items a user has added to their cart, including `user_id`, `items`, and other relevant data [S9](../database/catalog-db.md).

## Schema

The search telemetry system releases metrics in the format of JSON Lines (`traces.jsonl`, `metrics.jsonl`) stored in append-only files, with one record per line and no batching [S3](overview.md). The specific data structure or format of these metrics is not explicitly described in the provided sources.

Business metrics published by services are listed in their respective documentation, such as `order_checkout_outcomes_total` and `payment_amount_cents` [S1](overview.md).

Event logs from the catalog service can be used for monitoring interactions between the review service and the catalog service, as well as interactions with other services via the catalog API [S2](catalog.md). System metrics from the catalog service provide insights into its performance, such as CPU usage or memory consumption [S2](catalog.md) (further detail TBD). Request traces from the catalog service help identify bottlenecks in cart management processes [S3](overview.md).

The Search service can use event logs from the catalog service to monitor interactions between the review service and the catalog service. System metrics from the catalog service can provide insights into the performance of the catalog service. Request traces from the catalog service can help identify bottlenecks in cart management processes.

Note that the specific data stored in the cart database for each user is unclear (further detail TBD) [S4](../services/search.md).

## Open Questions

The meeting regarding the deployment and monitoring of the search engine's performance metrics resulted in the following outcomes:

* The Search service can use error logs from the catalog service to monitor interactions between the review service and the catalog service [S1](domain=sd, source=sd/services/search.md).
* System metrics from the catalog service provide insights into its performance, such as CPU usage or memory consumption [S2](domain=sd, source=sd/telemetry/catalog.md) (further detail TBD).
* Request traces from the catalog service help identify bottlenecks in cart management processes [S3](domain=sd, source=sd/telemetry/catalog.md).
* The Search service can use event logs from the catalog service to monitor interactions between the review service and the catalog service [S4](domain=sd, source=sd/telemetry/overview.md).
* System metrics from the catalog service can provide insights into its performance, such as CPU usage or memory consumption [S5](domain=sd, source=sd/telemetry/catalog.md) (further detail TBD).

The meeting also discussed implementing hit/miss tracking in seed cache to optimize data retrieval and reduce latency. This feature would help identify bottlenecks in cart management processes by analyzing request traces from the catalog service.

Note that the specific data stored in the cart database for each user is unclear (further detail TBD) [S4](domain=sd, source=sd/telemetry/catalog.md), but the Cart service stores information about items a user has added to their cart, including `user_id`, `items`, and other relevant data [S9](../database/catalog-db.md).
