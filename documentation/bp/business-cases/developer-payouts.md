# Business case: developer payouts

Pear Store is a marketplace; developers expect to be paid. This is
out of scope for the current implementation, but the system shape
already has hooks where a payout module would slot in.

## Where the money sits today

* Every successful purchase creates a `Payment` row in payment-svc with
  `amount_cents` and a provider `charge_id`.
* Every fulfilled order has a receipt in fulfillment-svc with the list
  of `app_ids` that were sold.

## What a payout module would do

A nightly job (or, more realistically, a periodic settlement task)
would:

1. Walk receipts since last settlement.
2. For each `app_id` in the receipts, look up the developer in catalog.
3. Aggregate amounts per developer, less the platform fee (TBD; the
   industry default of 30% is a starting point but is not encoded
   anywhere yet).
4. Issue a payout via a provider (ACH, SEPA, or a wallet platform —
   we'd add a hook similar to `stripe_hook` / `paypal_hook`).

## Problem

Developers and businesses face unique challenges related to payment processing, order fulfillment, and customer issue triage. One known gap is the handling of automatic refunds upon fulfillment failure [S3](catalog.md), which requires human intervention. Additionally, there is a pending question regarding what non-goals or out-of-scope work are explicitly excluded from this purchase data flow [S1](../runbooks/incident-response.md).

The system's architecture has been driven by modularity and decoupling of services, allowing for independent development and maintenance [S4](../../sd/services/payment.md). However, the exact duration of data retention for orders in the `failed` state is not explicitly stated [S6](../../sd/database/cart-db.md), which may be an area where SME input is required.

Refining the triage rules to better categorize and prioritize customer issues based on keywords could also benefit from SME input [S3](../../sd/services/review.md). Furthermore, improving payment processing by handling declined payments more efficiently (e.g., when a 402 error code is returned) [S4](../../sd/services/payment.md) and enhancing order fulfillment by automating refunds upon fulfillment failure are areas that require SME input.

## Stakeholders

Key stakeholders involved in the developer payouts feature include:

* Decision-makers:
	+ Catalog Service team, responsible for managing user carts and retrieving product information [S2](../../sd/services/payment.md)
	+ Account Service team, generating audit records, performance indicators, and distributed request data for monitoring and troubleshooting purposes [S4]
	+ Review Service team, emitting event logs related to interactions with the catalog service via the catalog API [S1](catalog-discovery.md)
* Funders:
	+ Not explicitly mentioned in the provided sources
* Other stakeholders:
	+ Search Service team, using system metrics from the catalog service to monitor its execution time, throughput, and latency [S2](../../sd/services/payment.md)
	+ Cart Service team, responsible for managing user carts and storing information about items a user has added to their cart [S9]

The payment process involves interactions between the order service, payment service, and fulfillment service. The key entities involved in this process are:

* **Payment**: owned by the payment service
* **Order**: handled by the order service
* **Cart**: managed by the cart service

These stakeholders are impacted by the developer payouts feature because they are responsible for managing user carts, retrieving product information, generating audit records, and monitoring system metrics. The payment process also involves interactions between these services, which may be affected by changes to the developer payouts feature.

Note that the specific roles and responsibilities of these stakeholders may not be fully defined in the provided sources.

## Success metrics

The developer payouts feature will be measured using the following metrics:

* `order_requests_total{status=success}`: The total number of successful order requests [S1](../../sd/telemetry/order.md).
* `order_request_duration_ms`: The average duration of order requests in milliseconds [S1](../../sd/telemetry/order.md).
* `order_checkout_outcomes_total{outcome=fulfilled}`: The total number of fulfilled checkout outcomes [S1](../../sd/telemetry/order.md).

Additionally, the following metrics related to payment processing will be used:

* `payment_authorize_outcomes_total`: The total number of authorize outcomes for payments [S7](../../sd/telemetry/payment.md).
* `payment_refund_outcomes_total`: The total number of refund outcomes for payments [S7](../../sd/telemetry/payment.md).
* `payment_amount_cents`: The histogram of payment amounts in cents [S7](../../sd/telemetry/payment.md).

These metrics will provide insights into the success rate, performance, and financial aspects of the developer payouts feature.

## Risks

Potential risks associated with implementing the developer payouts feature include:

* Inconsistent data validation: The system's reliance on manual intervention for automatic refunds upon fulfillment failure may lead to inconsistent data in the cart database, potentially causing issues with payment processing and order fulfillment [S1](../../sd/runbooks/pearcare-fraud.md).
* Triaging rules not optimized for keywords: The current triage rules may not accurately categorize and prioritize customer issues based on keywords, leading to potential delays or misallocation of resources [S3](../../sd/services/payment.md).
* Payment processing inefficiencies: Handling declined payments more efficiently is a pending question, which may lead to increased payment processing times and potential losses due to delayed refunds [S7](../../sd/services/cart.md).

Mitigation strategies that can be put in place include:

* Implementing data validation to ensure consistency with the cart database [S5](../../sd/runbooks/pearcare-fraud.md).
* Refining triage rules to better categorize and prioritize customer issues based on keywords [S3](../../sd/services/payment.md).
* Improving payment processing by handling declined payments more efficiently, potentially through automation of refunds upon fulfillment failure [S7](../../sd/services/cart.md).

Further detail regarding specific implementation details for automatic refunds upon fulfillment failure is TBD [S4](../../sd/services/fulfillment.md).

## Open questions

* **Refunds and chargebacks.** A payout has to net out refunds; today
  refunds are manual.
* **Tax forms.** 1099 / equivalents per region.
* **Currency.** Today everything is USD cents. Multi-currency catalog
  pricing is a follow-up.

## Why it isn't in this build

We chose to focus on the user-facing flow. Developer accounting is
its own product surface (developer portal, statements, dispute UX) and
would double the codebase without changing the storefront story. It
is on the roadmap.
