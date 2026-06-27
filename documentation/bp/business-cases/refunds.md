# Refunds

TODO: write this up.

There is a refund endpoint on the order service. It marks the order
refunded and tries to call payment.

We don't really have a policy yet. Probably 14 days?

For PearCare cancellations see the other doc.

## Overview

The current state of refunds in the system is that automatic refunds are not handled upon fulfillment failure, requiring human intervention to run a POST request to the Order Service (ORD) with the order ID for refund [S10](purchase-flow.md). This gap has been identified as a high-priority issue to address [S10](purchase-flow.md).

There is no business-level outcome counter for refunds [S1](../../sd/database/cart-db.md), and a new counter should be proposed. Additionally, on fulfillment failure, the order is currently left in `paid` state, and auto-rollback is a known gap [S3](../../sd/telemetry/payment.md). The expected behavior for a 4xx error on the `/refund` endpoint is that the order remains in its current state, as the refund process is best-effort and does not guarantee success [S4](../../sd/telemetry/payment.md).

The system's architecture has been driven by modularity and decoupling of services, allowing for independent development and maintenance [S4](../../sd/telemetry/payment.md). However, there are still areas where SME input is required to improve incident response processes.

To address the issue of automatic refunds not being handled upon fulfillment failure, it is necessary to automate this process. This can be achieved by implementing a new counter for refunds and proposing one [S1](../../sd/database/cart-db.md), as well as refining triage rules to better categorize and prioritize customer issues based on keywords [S3](../../sd/telemetry/payment.md).

Further detail regarding the specific data stored in the account database for each user is TBD [S7](../../sd/runbooks/incident-response.md). Additionally, determining the exact duration of data retention for orders in the `failed` state (further detail TBD) [S6](../../sd/services/cart.md), [S9](../../sd/services/account.md) is also an area that requires SME input.

## Problem

Customers or businesses face several problems regarding refunds, including:

Inconsistent data validation: The system's reliance on manual intervention for automatic refunds upon fulfillment failure may lead to inconsistent data in the cart database, potentially causing issues with payment processing and order fulfillment [S2](developer-payouts.md).

Handling declined payments more efficiently is a pending question, which may lead to increased payment processing times and potential losses due to delayed refunds [S7](../../sd/services/review.md).

Automatic refunds are not handled upon fulfillment failure, requiring human intervention to run a POST request to the Order Service (ORD) with the order ID for refund [S6](../../sd/services/fulfillment.md).

The exact duration of data retention for orders in the `failed` state is not explicitly stated [S6](../../sd/services/fulfillment.md), which may be an area where SME input is required.

To address these issues, this business case will:

Implement automatic refunds upon fulfillment failure to reduce manual intervention and potential inconsistencies in the cart database [further detail TBD].

Improve payment processing by handling declined payments more efficiently, potentially through automation of refunds upon fulfillment failure [S7](../../sd/services/review.md).

Enhance order fulfillment by automating refunds upon fulfillment failure.

Refine triage rules to better categorize and prioritize customer issues based on keywords [S3](../../sd/services/cart.md).

## Stakeholders

Key stakeholders involved in the refunds process include:

* Developers: responsible for handling automatic refunds upon fulfillment failure, which currently requires human intervention [S1](developer-payouts.md), [S2](../../sd/services/cart.md).
* Businesses: face unique challenges related to payment processing and order fulfillment, including handling declined payments more efficiently [S7](purchase-flow.md).
* Customer support teams: responsible for triaging customer issues based on keywords, but current rules may not accurately categorize and prioritize issues [S3](../../sd/services/account.md), [S10](developer-payouts.md).

Roles and responsibilities:

* Developers are responsible for automating refunds upon fulfillment failure, which is currently a manual process requiring human intervention [S2](../../sd/services/cart.md), [S11](developer-payouts.md).
* Businesses are responsible for handling declined payments more efficiently to improve payment processing times and reduce potential losses due to delayed refunds [S7](purchase-flow.md), [S10](developer-payouts.md).
* Customer support teams are responsible for refining triage rules to better categorize and prioritize customer issues based on keywords, but current rules may not accurately do so [S3](../../sd/services/account.md), [S10](developer-payouts.md).

Note: The exact duration of data retention for orders in the `failed` state is not explicitly stated, which may be an area where SME input is required [S6](pearcare-cancellations.md). Further detail regarding indexing strategies for the databases and backup strategies for the databases is TBD [S4](../../sd/database/cart-db.md), [S5](developer-payouts.md).

## Success metrics

To determine the effectiveness of the refunds process, the following metrics can be used:

* A new business-level outcome counter for refunds should be proposed to track the total number of refund outcomes (`order_refund_outcomes_total`) [S1](../../sd/telemetry/order.md).
* The average time it takes to process a refund, which can be tracked using performance metrics such as fulfillment time and payment processing speed [S4](purchase-flow.md).
* The percentage of successful refunds compared to the total number of refund attempts, which can be calculated using conversion rates (e.g., `order_requests_total{status=success}`) [S4](purchase-flow.md).

To track these metrics, the system should be able to break down declined payments by provider (e.g., Stripe, PayPal, PearCard), and a new counter for refunds should be implemented to provide per-item granularity when catalog is slow on a specific app [S1](../../sd/telemetry/order.md). Additionally, the system should be able to automatically handle refunds upon fulfillment failure, which is currently not handled automatically and requires human intervention [S2](../../sd/services/cart.md).

Further detail regarding the specific data stored in the account database for each user is TBD [S7](developer-payouts.md).

## Risks

Potential risks associated with implementing a refund policy include:

* Inconsistent data validation: The system's reliance on manual intervention for automatic refunds upon fulfillment failure may lead to inconsistent data in the cart database, potentially causing issues with payment processing and order fulfillment [S1](developer-payouts.md).
* Triaging rules not optimized for keywords: The current triage rules may not accurately categorize and prioritize customer issues based on keywords, leading to potential delays or misallocation of resources [S3](../../sd/runbooks/incident-response.md).
* Payment processing inefficiencies: Handling declined payments more efficiently is a pending question, which may lead to increased payment processing times and potential losses due to delayed refunds [S7](pearcare-claim-economics.md).

To mitigate these risks:

1. **Improve transparency**: Provide clear documentation and access to pricing math, triage rules, and coverage windows for the storefront team.
2. **Standardize data consistency**: Define a clear consistency model and ensure that all services use it consistently.
3. **Monitor claim lifecycle changes**: Carefully review any changes to the claim lifecycle process or integration with PearCare Claim service.

Additionally, implementing measures such as:

* Implementing a cooling-off period
* Offering prorated refunds
* Implementing a minimum commitment period

may help mitigate risks associated with refund policies [S10](pearcare-cancellations.md). However, further detail is needed regarding specific data stored in the account database for each user (further detail TBD) [S12].

Note: The exact duration of data retention for orders in the `failed` state is not explicitly stated, which may be an area where SME input is required [S6](../../sd/services/fulfillment.md) and [S4](../../sd/services/cart.md).

## Open Questions

Subject matter experts (SMEs) need to address several concerns regarding refunds. Specifically:

* Automatic refunds upon fulfillment failure are not handled and require human intervention [S1](../../sd/services/fulfillment.md), [S2](../../sd/runbooks/incident-response.md), [S3](purchase-flow.md), [S4](../../sd/services/account.md), [S5](../../sd/services/cart.md), [S6](../../sd/pearcare/integration.md).
* The exact duration of data retention for orders in the `failed` state is not explicitly stated [S4](../../sd/services/account.md), [S6](../../sd/pearcare/integration.md), [S9](../../sd/services/fulfillment.md).
* Refining triage rules to better categorize and prioritize customer issues based on keywords is necessary [S3](purchase-flow.md), [S5](../../sd/services/cart.md).

Further detail is needed regarding the specific data stored in the account database for each user [S7](../../sd/telemetry/payment.md), [S12].
