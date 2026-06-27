# pearcare cancellations

Users can cancel their plan from the account page (or via API).

We don't refund anything when they cancel. Should we? Maybe pro-rata.

The status flips to `cancelled`. Past claims are unaffected.

## Problem

The user or business problem that the PearCare cancellations feature addresses is related to warranty plans and claims. According to [S1](../../sd/pearcare/overview.md), PearCare lets a user buy a protection plan against an app they own, and lets them file claims when something goes wrong. This suggests that the primary issue being addressed by PearCare is the management of warranty-related issues for users.

The PearCare cancellations feature likely addresses the business problem of managing warranty plans and claims independently of the storefront's order processing and fulfillment systems [S5](../../sd/pearcare/overview.md). By having its own sub-system, PearCare can evolve and be deployed in a way that is independent of the rest of the store.

The cancellations feature also relates to the successful issue resolution KPI mentioned in [S3](../../sd/architecture/overview.md), as it indicates that users are able to file claims and have them resolved through PearCare. This suggests that the system is able to categorize and prioritize customer issues based on keywords, which is an important aspect of managing warranty-related issues.

The key stakeholders affected by the PearCare cancellations feature are:

* Customers who purchase a protection plan through PearCare [S1](../../sd/pearcare/overview.md)
* Product managers responsible for managing the warranty product and its integration with other services [S2](../../sd/pearcare/overview.md)

The cancellations feature is likely used to measure the percentage of users who have purchased a warranty plan or filed a claim through PearCare, which can be used to evaluate the effectiveness of PearCare in addressing user concerns and providing protection against app-related issues.

Note that further detail regarding specific data stored in the account database for each user is TBD [S9](../../sd/telemetry/pearcare-claim.md).

## Stakeholders

Beneficiaries of a cancelled PearCare policy are users who have purchased a protection plan through PearCare [S5](pearcare-attach-rate.md). 

The stakeholders with authority over cancellation decisions include product managers responsible for managing the warranty product and its integration with other services [S2](../../sd/architecture/overview.md), as well as customers who purchase a protection plan through PearCare [S1](../../sd/runbooks/pearcare-fraud.md).

Note that the exact process of cancelling a PearCare policy is not explicitly stated in the provided sources.

## Success metrics

To measure the success of implementing this business case, we will use a combination of metrics from various sources. The developer payouts feature will be measured using the following metrics:

* Success rate: This metric is not explicitly mentioned in any source, but it can be inferred as a key performance indicator (KPI) for the developer payouts feature [S3](developer-payouts.md).
* Performance: Metrics such as fulfillment time and payment processing speed will provide insights into the performance of the developer payouts feature [S7](developer-payouts.md).
* Financial aspects: Metrics related to payment processing, such as handling declined payments more efficiently, will provide insights into the financial aspects of the developer payouts feature [S6](developer-payouts.md).

Additionally, we will use metrics from the telemetry system to monitor the success of implementing this business case. These metrics include:

* Disaster recovery procedures [S4](../../sd/telemetry/overview.md).
* Data retention policies for orders in the `failed` state (further detail TBD) [S2](../../sd/telemetry/overview.md).
* Per-app instrumentation inside `/fulfill` for the fulfillment service [S7](developer-payouts.md).
* Counter for receipt reads vs library reads for the fulfillment service [S7](developer-payouts.md).
* Aggregation of metrics by app_id for the fulfillment service, which requires going to the database instead of relying on metrics [S7](developer-payouts.md).

It is worth noting that some of these metrics require SME input to refine and improve their effectiveness.

## Risks

Potential risks associated with allowing users to cancel their pearcare plans include:

* Loss of revenue: If users can cancel their plans at any time, it may lead to a loss of revenue for the company, as users may not be committed to purchasing the warranty product [S1](../../sd/pearcare/overview.md).
* Decreased customer satisfaction: Canceling a plan may lead to decreased customer satisfaction, as users may feel that they have invested in a product only to have it canceled without receiving any benefits [S2](../../sd/pearcare/overview.md).
* Increased administrative burden: Allowing users to cancel their plans at any time may increase the administrative burden on the company, as staff will need to handle cancellations and refunds [S3](../../sd/pearcare/plan-service.md).

To mitigate these risks, the company could consider implementing the following measures:

* Implement a cooling-off period: This would allow users to cancel their plans within a certain timeframe (e.g. 30 days) without penalty, but after that, cancellation fees or penalties could be applied [further detail TBD].
* Offer prorated refunds: If a user cancels their plan, the company could offer a prorated refund based on the time remaining on the plan [S3](../../sd/pearcare/plan-service.md).
* Implement a minimum commitment period: This would require users to commit to purchasing the warranty product for a certain amount of time (e.g. 6 months) before they can cancel their plan [further detail TBD].

Note that these measures are not explicitly stated in the sources, but are potential solutions based on the information provided.

## Open Questions

There are several areas that require SME input to improve the incident response process and resolve open questions:

* Refining triage rules to better categorize and prioritize customer issues based on keywords [S3](../../sd/services/fulfillment.md) (e.g., refining triage rules to better categorize and prioritize customer issues based on keywords could also benefit from SME input [S9](developer-payouts.md)).
* Improving payment processing by handling declined payments more efficiently (e.g., when a 402 error code is returned) [S4](../../sd/database/payment-db.md).
* Enhancing order fulfillment by automating refunds upon fulfillment failure.
* Clarifying the consistency model, specific data stored in the cart database, and data stored in the account database [S1](../../sd/database/review-db.md), [S3](../../sd/services/fulfillment.md), [S5](../../sd/database/review-db.md), [S8](../../sd/database/catalog-db.md), [S11](catalog-discovery.md).
* Determining the exact duration of data retention for orders in the `failed` state (further detail TBD) [S6](../../sd/database/catalog-db.md), [S9](developer-payouts.md).

Additionally, there are areas where further detail is TBD regarding:

* Indexing strategies for the databases [S4](../../sd/database/payment-db.md)
* Backup strategies for the databases [S5](../../sd/database/review-db.md)
* The exact duration of data retention for orders in the `failed` state (further detail TBD)

The unresolved issues or areas that require SME input include:

* Triage rules, including eligibility criteria and required tiers [S15](../../sd/architecture/data-flow-pearcare-claim.md).
* Clarifying the consistency model and specific data stored in the cart database [S1](../../sd/database/review-db.md), [S3](../../sd/services/fulfillment.md), [S5](../../sd/database/review-db.md), [S8](../../sd/database/catalog-db.md), [S11](catalog-discovery.md).

Note: The above list is not exhaustive, as there may be other areas that require SME input.
