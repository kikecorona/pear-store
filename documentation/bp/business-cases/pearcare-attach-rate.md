# Business case: PearCare attach rate

## What "attach rate" means

The fraction of paid app purchases that include a PearCare plan.

```
attach_rate = orders_with_at_least_one_enrollment / paid_orders_with_eligible_app
```

The denominator only counts orders containing **eligible** apps —
i.e. apps that have entries in `APP_PLAN_MAP`. Computing attach rate
against the full storefront would conflate "we don't offer plans on
this app" with "the user said no", and would understate the metric
for cases like "user bought one eligible and three ineligible apps in
one cart".

## Why this matters

Warranty programs have a small set of inputs and three outputs that
matter: attach rate, claim rate, and average claim cost. Of those,
attach rate is the only one we can directly influence with product
changes. Claim rate is mostly a function of the underlying app
quality.

A 1-percentage-point swing in attach rate on premium apps materially
moves total PearCare revenue, because the SKUs are priced at 5–25%
of the underlying app price. We expect the curve to be steepest at
the app detail page, which is the only surface where the user
actually sees the offer.

## Levers we have today

* **Default presentation.** The detail page renders plans inline with
  the buy button. We currently default to "No protection" — explicit
  opt-in. Switching to a recommended-default would lift attach rate
  but raises a customer-trust question; we have not made that change.
* **Plan ladder shape.** Today eligible apps offer either a 3-rung
  (basic-12 / plus-12 / loss-12) or 2-rung (basic-12 / basic-24)
  ladder. Decoy effect studies suggest 3-rung outperforms 2-rung even
  if the middle SKU sells less than 5% of the time.
* **Eligible-app expansion.** Each app added to `APP_PLAN_MAP` adds
  surface area. We have been conservative: only premium / pro apps
  qualify today.

## Levers we don't have yet

* Post-purchase plan offer. Apple's "you can add AppleCare within 60
  days" model. Would require a new endpoint on plan-svc.
* Reminders close to expiry. No notification service exists yet.

## Pitfalls

* **Plan visibility on free apps.** A free app showing a $20 plan would
  be confusing. The `APP_PLAN_MAP` deliberately excludes free apps;
  do not relax this without a UX review.
* **Hidden plan churn.** Monthly billing (`plan.plus.month`) creates
  cancellation surface; see `pearcare-cancellations.md`.

## Problem

The user or business problem that the PearCare attach rate addresses is related to warranty plans and claims. According to [S2](../../sd/pearcare/overview.md), PearCare lets a user buy a protection plan against an app they own, and lets them file claims when something goes wrong. This suggests that the primary issue being addressed by PearCare is the management of warranty-related issues for users.

The PearCare attach rate likely measures the percentage of users who have purchased a warranty plan or filed a claim through PearCare. This metric can be used to evaluate the effectiveness of PearCare in addressing user concerns and providing protection against app-related issues.

In terms of specific business problems, PearCare addresses the issue of managing warranty plans and claims independently of the storefront's order processing and fulfillment systems [S5](../../sd/pearcare/overview.md). By having its own sub-system, PearCare can evolve and be deployed independently of the rest of the store, which is important for the first few months a warranty product is live.

The attach rate also relates to the successful issue resolution KPI mentioned in [S3](../../sd/architecture/overview.md), as it indicates that users are able to file claims and have them resolved through PearCare. This suggests that the system is able to categorize and prioritize customer issues based on keywords, which is an important aspect of managing warranty-related issues.

Overall, the PearCare attach rate addresses the business problem of managing warranty plans and claims for users, and provides a metric for evaluating the effectiveness of PearCare in addressing user concerns.

## Stakeholders

The key stakeholders affected by the PearCare attach rate are:

* Customers who purchase a protection plan through PearCare [S1](../../sd/pearcare/overview.md)
* Product managers responsible for managing the warranty product and its integration with other services [S2](../../sd/pearcare/overview.md)
* The sales team, as an increase in PearCare attach rate may impact their sales strategies and revenue projections [further detail TBD]

Note that the Fulfillment Service plays a crucial role in processing replacement entitlements and claims, making it a key stakeholder in the PearCare attach rate [S3](../../sd/database/pearcare-claim-db.md).

## Success metrics

The key performance indicators (KPIs) that will be used to measure the success of the PearCare attach rate are:

* Time-to-first-detail-page-view (T1) — the dominant funnel metric [S2](catalog-discovery.md)
* Search-to-detail conversion (search results clicked / searches) [S2](catalog-discovery.md)
* Top-chart click-through rate [S2](catalog-discovery.md)

These KPIs are related to user cart management and interactions with the catalog service, which is responsible for managing user carts and retrieving product information. The account service generates performance indicators related to user cart management, including metrics such as the number of carts created or updated by the catalog service [S6](../../sd/services/account.md).

The telemetry system analyzes transmitted data to provide real-time monitoring and insights, including analyzing event logs, system metrics, and request traces to identify bottlenecks in cart management processes [S3](../../sd/telemetry/README.md). The catalog service generates event logs related to interactions with other services via the catalog API [S1](../../sd/telemetry/overview.md), which can be used for monitoring interactions between services.

Note that specific performance indicators and distributed request data generated by the account service are TBD [S6](../../sd/services/account.md).

## Risks

Key risks associated with improving the PearCare attach rate include:

1. **Lack of transparency**: The storefront does not have access to pricing math, triage rules, or coverage windows, which are owned by PearCare [S12](../../sd/runbooks/pearcare-fraud.md). This lack of visibility can lead to misaligned incentives and potential fraud.
2. **Data inconsistencies**: The frontend trusts data returned from the plan service, but the consistency model and specific data stored in the cart database are unclear [S12](../../sd/runbooks/pearcare-fraud.md) (further detail TBD). Inconsistent data can lead to errors and mistrust between services.
3. **Claim lifecycle ownership**: The claim lifecycle (`filed → triaged → approved | denied`) is owned by the PearCare Claim service, which determines the resolution type and triggers corresponding hooks [S2](../../sd/runbooks/pearcare-fraud.md), [S12](../../sd/runbooks/pearcare-fraud.md). Improper integration or changes to this process can lead to errors or security vulnerabilities.

To mitigate these risks:

1. **Improve transparency**: Provide clear documentation and access to pricing math, triage rules, and coverage windows for the storefront team.
2. **Standardize data consistency**: Define a clear consistency model and ensure that all services use it consistently.
3. **Monitor claim lifecycle changes**: Carefully review any changes to the claim lifecycle process or integration with PearCare Claim service.

Note: Further detail regarding specific data stored in the account database for each user is TBD [S9](../../sd/telemetry/pearcare-claim.md).

## Open Questions

Remaining unanswered questions or uncertainties related to the PearCare attach rate business case include:

* The exact duration of data retention for orders in the `failed` state [S10](../runbooks/pearcare-fraud.md) (further detail TBD).
* The specific data stored in the account database for each user, particularly with regards to cart information and order status [S5](../database/pearcare-claim-db.md), [S3](../database/pearcare-claim-db.md) (further detail TBD).
* The consistency model and specific data stored in the cart database, which may lead to data inconsistencies [S9](../runbooks/pearcare-fraud.md) (further detail TBD).

These uncertainties highlight areas where further investigation or clarification is needed to fully understand the PearCare attach rate business case.
