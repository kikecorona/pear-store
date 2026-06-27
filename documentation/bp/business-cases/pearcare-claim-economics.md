# Business case: PearCare claim economics

PearCare is profitable when premiums collected exceed claim costs over
the term of a plan. The breakdown matters more than the headline.

## Claim cost by resolution path

| Path        | Cost we incur | Notes |
| ----------- | ------------- | ----- |
| support     | small (CS time) | A few hundred per 1k claims at most. |
| repair      | medium        | Vendor flat fees. Set in vendor contracts; not in this codebase. |
| replacement | full app price | We mint a license; no money to a vendor, but the app developer is paid for the replacement (per `developer-payouts.md` — that part is not built yet). |

## What the auto-triage does to economics

The default `_auto_triage` rule biases towards `support`. That is the
right default because support claims are nearly free and the alternative
is a real-money outflow on a claim that should never have qualified.

The risk is the other direction: a real damage claim that the operator
or the rule classifies as `support` and therefore underpays the user.
We do not currently track customer-satisfaction signals on claim
resolution. A reasonable next step would be a per-claim follow-up
rating.

## Open issues

* No expiry check at file-time. A user could file against an expired
  enrollment and we would happily approve it.
* No cap on per-enrollment claim count. PearCare+ promises "unlimited
  claims" but in practice this needs a soft limit + manual review
  threshold.
* Vendor cost is not stored anywhere in our system; we cannot compute
  margin without joining against an external billing report.

## Stakeholders

The key stakeholders involved in the PearCare claim economics business case are:

* Customers who purchase a protection plan through PearCare [S1](../../sd/pearcare/overview.md)
* Product managers responsible for managing the warranty product and its integration with other services [S2](../../sd/pearcare/overview.md)
* The sales team, as an increase in PearCare attach rate may impact their sales [S2](../../sd/pearcare/overview.md)

These stakeholders are affected by the PearCare attach rate, which measures the percentage of users who have purchased a warranty plan or filed a claim through PearCare. This metric can be used to evaluate the effectiveness of PearCare in addressing user concerns and providing protection against app-related issues [S2](../../sd/pearcare/overview.md).

## Success metrics

The metrics used to determine the success of the PearCare claim economics business case include:

* Attach rate: The fraction of paid app purchases that include a PearCare plan, calculated as `orders_with_at_least_one_enrollment / paid_orders_with_eligible_app` [S3](pearcare-attach-rate.md).
* Claim rate: The percentage of users who have filed a claim through PearCare (further detail TBD) [S2](pearcare-attach-rate.md).

These metrics are used to evaluate the effectiveness of PearCare in addressing user concerns and providing protection against app-related issues.

## Risks

Key risks associated with the PearCare claim economics business case include:

* Loss of revenue: If users can cancel their plans at any time, it may lead to a loss of revenue for the company, as users may not be committed to purchasing the warranty product [S3](../../sd/pearcare/overview.md).
* Decreased customer satisfaction: Canceling a plan may lead to decreased customer satisfaction, as users may feel that they have invested in a product only to have it canceled without receiving any benefits [S2](../../sd/pearcare/overview.md).
* Increased administrative burden: Allowing users to cancel their plans at any time may increase the administrative burden on the company, as staff will need to handle cancellations and refunds [S3](../../sd/pearcare/plan-service.md).

To mitigate these risks, the company could consider implementing a cooling-off period (further detail TBD), offering prorated refunds based on the time remaining on the plan [S3](../../sd/pearcare/plan-service.md), or implementing a minimum commitment period (further detail TBD).

## Open Questions

The remaining open questions or areas of uncertainty related to the PearCare claim economics business case include:

* Clarifying the consistency model and specific data stored in the cart database [S1](../database/catalog-db.md)
* Understanding the data stored in the account database for each user, particularly with regards to cart information and order status [S5](../database/pearcare-claim-db.md), [S3](../database/pearcare-claim-db.md) (further detail TBD)
* The transparency of pricing math, triage rules, or coverage windows, which are owned by PearCare but not accessible to the storefront [S9](../runbooks/pearcare-fraud.md)

These areas require further investigation and SME input to fully understand the implications for the business case.
