# ratings trust

Reviews show up on the app detail page. Ratings drive the top charts.

We don't check if the reviewer owns the app. We probably should.

## Problem

Unresolved business issue: Unverified app ownership in review sections can lead to unauthorized claims being filed against a user's warranty plan, potentially resulting in replacement licenses being issued without proper authorization.

When a claim resolves to a replacement, the Fulfillment Service (FUL) processes the order and interacts with the account database to verify the user's identity [S2](../../sd/pearcare/overview.md). However, if the app ownership is unverified, it may not be possible to accurately determine whether the claimant has a valid warranty plan for the affected app. This can lead to unauthorized claims being filed and replacement licenses being issued without proper authorization.

The data flow diagram shows that the CL service sends a request to the FUL service with the order ID and claim ID for replacement resolutions [S3], [S4]. However, if the app ownership is unverified, it may not be possible to accurately determine whether the claimant has a valid warranty plan for the affected app.

The services involved in this process are:

* PearCare Claim (CL): handles claim filing and resolution determination
* Fulfillment Service (FUL): processes replacement orders

Note that further detail regarding the specific data stored in the account database for each user is TBD [further detail TBD].

## Stakeholders

The stakeholders affected by the issue of fake or manipulated ratings include:

* Customers who have purchased a protection plan through PearCare [S1](../../sd/runbooks/pearcare-fraud.md), as they are directly impacted by the accuracy of ratings [further detail TBD]
* Product managers responsible for managing the warranty product and its integration with other services [S2](piracy-and-licensing.md), as they need to make informed decisions based on accurate ratings
* The sales team, as an increase in PearCare attach rate may impact their sales [S5](pearcare-claim-economics.md)
* Decision-makers involved in the purchase flow, including:
	+ The Catalog Service team, responsible for managing user carts and retrieving product information [S6](purchase-flow.md)
	+ The Account Service team, generating audit records, performance indicators, and distributed request data for monitoring and troubleshooting purposes [S4](pearcare-cancellations.md)
	+ The Review Service team, emitting event logs related to interactions with the catalog service via the catalog API [S1](../../sd/runbooks/pearcare-fraud.md)

Funders affected by this issue are not explicitly mentioned in the provided sources.

## Success metrics

To measure success in addressing the issue of fake or manipulated ratings, such as increased user trust or reduced rating manipulation, we can consider the following metrics:

1. Reduction in rating manipulation: The review service's ability to detect and prevent fake reviews can be measured by tracking the number of reviews that are flagged as suspicious or removed due to manipulation [S2](../../sd/services/review.md). This can be done by analyzing the data stored in the catalog database, which contains schemas for managing customer carts and orders, as well as retrieving product information from the catalog service [S5].
2. Increase in user trust: User trust can be measured by tracking changes in user behavior, such as increased engagement with reviews or ratings [further detail TBD]. The review service's storage and processing functions are likely tied to the catalog database, which contains schemas for managing customer carts and orders, as well as retrieving product information from the catalog service [S5].
3. Improved rating accuracy: The review service can be designed to provide more accurate ratings by incorporating features such as weighted averages or machine learning algorithms that take into account user behavior and other factors [further detail TBD]. The specific data stored in the account database for each user is not specified, but it likely involves a JSON object containing the rating and free-text review [S3](../../sd/services/review.md).

To measure these metrics, we can use the following endpoints:

* GET /reviews/<app_id>: This endpoint provides all reviews for a given app, which can be used to analyze user behavior and identify potential issues with ratings [S2](../../sd/services/review.md).
* POST /apps/<app_id>/rating: This endpoint is used by the review service to initiate a review transaction, which involves updating aggregate ratings in the catalog database [S3](../../sd/services/review.md).

Note that further detail on the specific storage and processing mechanisms used by the review service is TBD.

## Risks

Potential risks associated with addressing fake or manipulated ratings include:

The inability to track when a row was last updated in the catalog database, making it difficult to detect and prevent tampering [S6](../../sd/database/catalog-db.md). Anyone who can reach the catalog API can clobber an app's average rating without validation [S6](../../sd/database/catalog-db.md), which could be exploited by PearCare scams that target vulnerabilities in the integration between the storefront and PearCare services [S8](../../sd/runbooks/pearcare-fraud.md).

To mitigate these risks, the storefront should implement rate limiting on the rating endpoint to prevent any caller from clobbering an app's average rating [S9](../../sd/services/catalog.md). Additionally, including details about the primary characteristics of PearCare scams and their associated vulnerabilities in a runbook could help identify potential entry points for attackers [S4](../../sd/runbooks/pearcare-fraud.md) (further detail TBD).

## Open Questions

There are several unresolved questions and areas that need SME input to further develop this business case.

The Catalog service has open questions or areas that need SME input, including [S1](../../sd/services/catalog.md). The payment database requires SME input regarding data retention and durability [S7](../../sd/database/review-db.md).

The incident response process has several areas where SME input is required, including refining triage rules to better categorize and prioritize customer issues based on keywords [S3](../../sd/runbooks/pearcare-fraud.md), improving payment processing by handling declined payments more efficiently [S4](../../sd/database/payment-db.md), and enhancing order fulfillment by automating refunds upon fulfillment failure.

Additionally, there are open questions regarding the consistency model, specific data stored in the cart database, and data stored in the account database [S1](../../sd/services/catalog.md), [S5](../../sd/database/catalog-db.md). The exact duration of data retention for orders in the `failed` state is not explicitly stated [S6](../../sd/services/fulfillment.md).

Furthermore, SME input is required to clarify indexing strategies for the databases [S4](../../sd/database/payment-db.md) and backup strategies for the databases [S5](../../sd/database/catalog-db.md).
