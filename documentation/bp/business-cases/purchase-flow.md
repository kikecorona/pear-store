# Business case: the purchase flow

## Goal

A signed-in user with an app in their cart can press one button and end
up with that app in their library. Every step that asks them to type or
re-confirm something is a leak in the funnel.

## Why one orchestrator

The purchase flow is the place where retries, partial failures, and
"are we paid yet?" all collide. We made an early call: only one
service — order-svc — knows the full shape. Cart-svc never speaks to
payment. Fulfillment never speaks to cart. The payment service does
not know what an order is in any deep sense; it just knows there is an
amount and a token.

The benefit is operational: when something is wrong with a checkout,
there is exactly one service to look at. The cost is that order-svc is
the single most important service in the system to keep healthy.

## Money in, license out

Concretely the flow is:

1. The user presses *Check out*. The frontend posts to order-svc.
2. order-svc reads the cart, prices it from catalog, and creates a
   pending order.
3. order-svc authorizes payment. Decline → the order is preserved as
   `failed` for audit, and the frontend shows the provider error.
4. order-svc calls fulfillment, which mints licenses and per-platform
   download URLs.
5. order-svc clears the cart. The frontend redirects to the receipt page.

## Money lost windows

* **Payment authorized but fulfillment failed.** The user is charged
  and the order sits in `paid`. *Today this requires a manual
  refund* — see `../../sd/runbooks/incident-response.md`. Closing this gap is
  the highest-priority follow-up.
* **Catalog price drift.** If a price changes between "added to cart"
  and "checkout", we charge the new price. We chose this trade-off
  because the alternative — pricing at cart-add time — would mean
  carrying state in cart-svc that has to expire. Acceptable for now.

## Future enhancements

* Idempotent checkout. POST `/checkout` with the same `Idempotency-Key`
  should return the same order id, even on a retry. This is needed
  before we put a button-mashing user behind a flaky network.
* Saved payment methods on the account record. Right now every
  authorize call synthesizes a token.
* Receipt email via a future notification service.

## Stakeholders

The purchase flow impacts various teams and departments within the organization. Decision-makers involved in this process include:

* The Catalog Service team, responsible for managing user carts and retrieving product information [S2](../../sd/services/payment.md)
* The Account Service team, generating audit records, performance indicators, and distributed request data for monitoring and troubleshooting purposes [S4](../../sd/architecture/data-flow-purchase.md)
* The Review Service team, emitting event logs related to interactions with the catalog service via the catalog API [S1](catalog-discovery.md)

Other stakeholders impacted by the purchase flow include:

* The Search Service team, using system metrics from the catalog service to monitor its execution time, throughput, and latency [S2](../../sd/services/payment.md)
* The Cart Service team, responsible for managing user carts and storing information about items a user has added to their cart [S9]
* The Order Service team, which retrieves the cart contents from the cart service when a user initiates a checkout [S3](../architecture/data-flow-purchase.md)
* The Fulfillment Service team, responsible for minting licenses and per-platform download URLs [S2](../architecture/overview.md)

These teams interact with each other during the purchase process, including during checkout, where the order service retrieves cart contents from the cart service, and the fulfillment service is responsible for processing the transaction [S3](../../sd/database/cart-db.md).

## Success metrics

To assess the efficiency and success of the checkout process, the following metrics will be utilized:

* Conversion rates (e.g., `order_requests_total{status=success}`, `order_checkout_outcomes_total{outcome=fulfilled}`) [S9](developer-payouts.md)
* Average order values (e.g., `payment_amount_cents`) [S2](../../sd/telemetry/overview.md)
* Fulfillment time and payment processing speed (performance metrics) [S7](developer-payouts.md)
* Handling declined payments more efficiently (financial aspect metric) [S6](catalog-discovery.md)

These metrics will provide insights into the performance of the checkout process, including its efficiency in handling successful orders, average order values, and financial aspects such as payment processing.

## Risks

Potential risks associated with implementing and maintaining the purchase flow include:

* Loss of revenue due to users canceling their plans at any time, potentially leading to decreased customer satisfaction [S3](../../sd/architecture/overview.md).
* Decreased customer satisfaction resulting from canceled plans without receiving benefits [S2](pearcare-attach-rate.md).
* Increased administrative burden on the company due to allowing users to cancel plans at any time [S3](../../sd/architecture/overview.md).
* Inconsistent data stored in the cart database, which can lead to errors and mistrust between services [S6].
* Improper integration or changes to the claim lifecycle process, potentially leading to errors or security vulnerabilities [S2](pearcare-attach-rate.md), [S12].

To mitigate these risks:

1. **Improve transparency**: Provide clear documentation and access to pricing math, triage rules, and coverage windows for the storefront team.
2. **Standardize data consistency**: Define a clear consistency model and ensure that all services use it consistently.
3. **Monitor claim lifecycle changes**: Carefully review any changes to the claim lifecycle process or integration with PearCare Claim service.

Note: Further detail regarding specific data stored in the account database for each user is TBD [S9].

## Open Questions

The following specific open questions or areas of uncertainty need SME input to resolve:

* Clarifying what non-goals or out-of-scope work are explicitly excluded from this purchase data flow [S1](catalog.md) and [S5](../runbooks/incident-response.md).
* Handling automatic refunds upon fulfillment failure, which requires human intervention [S2](catalog.md), [S6](../architecture/data-flow-purchase.md), and [S11](../database/catalog-db.md).
* Refining the triage rules to better categorize and prioritize customer issues based on keywords [S3](../runbooks/pearcare-fraud.md) and [S5](../architecture/overview.md).
* Improving payment processing by handling declined payments more efficiently.
* Enhancing order fulfillment by automating refunds upon fulfillment failure.
* Indexing strategies for the databases (further detail TBD) [S4](../database/review-db.md).
* Backup strategies for the databases (further detail TBD) [S5](../database/review-db.md).
* Index and backup strategies (further detail TBD) [S5](../database/review-db.md).
* The exact duration of data retention for orders in the `failed` state is not explicitly stated, which may be an area where SME input is required [S6](../database/payment-db.md) and [S4](../pearcare/overview.md).
