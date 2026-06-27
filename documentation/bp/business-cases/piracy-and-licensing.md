# piracy and licensing

A licens is what proves you bought an app. We mint one per app per order
in the fulfilment service. They are perpetual which means once minted they
never expire. Which is a problem if the user refunds becuase the licens stays
valid.

There is no DRM in this system. The download URLs are signed but the binarys
themselves don't check the licens, that is the OS's job. If somebody pulls a
URL out of the receipt and shares it with a freind, the freind can install too.
That's a known gap, the URL has expires_at but its not enforced anywhere yet.

For PearCare replacements, we mint a *second* licens for the same user, same
app. The user technically has two now. Some folks think this is a bug, its not.
The data model says one user can hold many licenses for the same app and that
makes the family share story easier later.

If we wanted to actually fight piracy, the steps would be:

- enforce expires_at on the CDN
- have a license check API that the running app pings
- revoke licenses on refund

None of that is built. The system mostly assumes good actors. Anyone trying
to pirate apps via this store is going to have a great time.

## Problem

The core pain points in copyright infringement and intellectual property management that this business case seeks to resolve are:

* Loss of revenue due to users canceling their plans at any time, potentially leading to decreased customer satisfaction [S3](pearcare-claim-economics.md).
* Decreased customer satisfaction resulting from canceled plans without receiving benefits [S2](developer-payouts.md).
* Increased administrative burden on the company due to allowing users to cancel plans at any time [S3](pearcare-claim-economics.md).
* Inconsistent data stored in the cart database, which can lead to errors and mistrust between services [S6](pearcare-attach-rate.md).
* Improper integration or changes to the claim lifecycle process, potentially leading to errors or security vulnerabilities [S2](developer-payouts.md), [S12].

The business case aims to address these pain points by improving transparency, standardizing data consistency, and monitoring claim lifecycle changes.

## Stakeholders

Key stakeholders involved in addressing piracy and licensing issues in this system include:

Decision-makers:
* Catalog Service team, responsible for managing user carts and retrieving product information [S2](developer-payouts.md)
* Account Service team, generating audit records, performance indicators, and distributed request data for monitoring and troubleshooting purposes [S4](pearcare-cancellations.md)
* Review Service team, emitting event logs related to interactions with the catalog service via the catalog API [S1](developer-payouts.md)

Other stakeholders:
* Search Service team, using system metrics from the catalog service to monitor its execution time, throughput, and latency [S2](developer-payouts.md)
* Cart Service team, responsible for managing user carts and storing information about items a user has added to their cart [S9]

## Success metrics

To measure the effectiveness of addressing piracy and licensing issues in this system, several metrics or indicators can be used. Specifically:

* The number of licenses issued (`fulfillment_licenses_issued_total`) [S1](../../sd/telemetry/overview.md)
* The percentage of users who have purchased a warranty plan or filed a claim through PearCare (PearCare attach rate) [S3](pearcare-attach-rate.md), which addresses the business problem of managing warranty plans and claims for users
* The ratio of receipt reads to library reads for the fulfillment service, which can indicate the effectiveness of licensing and piracy prevention measures [S2](../../sd/telemetry/overview.md)

These metrics can be used to evaluate the effectiveness of addressing piracy and licensing issues in this system.

## Risks

**Risks and Challenges**

Addressing piracy and licensing issues in this system poses several risks and challenges. The Fulfillment Service (FUL) plays a key role in managing entitlements and replacements, but it has vulnerabilities associated with PearCare scams [S9](../../sd/services/fulfillment.md). These scams exploit weaknesses in the integration between the storefront and PearCare services [S8](../../sd/runbooks/pearcare-fraud.md).

The primary characteristics of these scams include deterministic download URLs that are not actually served by any service [S3](../../sd/services/fulfillment.md), lack of revocation mechanisms (cancelling or refunding does not invalidate the license) [S3](../../sd/services/fulfillment.md), and an `expires_at` field on download URLs that is not enforced anywhere [S3](../../sd/services/fulfillment.md). To mitigate these vulnerabilities, implementing data validation to ensure consistency with the cart database and refining triage rules to better categorize and prioritize customer issues based on keywords are recommended [S7](developer-payouts.md).

Additionally, the system lacks real authentication mechanisms (no OAuth, no JWT, no rate limiting) [S11](../../sd/architecture/overview.md), which could make it vulnerable to unauthorized access. The lack of observability features (metrics, traces, or structured logs) also makes it difficult to monitor and detect potential issues [S11](../../sd/architecture/overview.md).

## Open Questions

The remaining open questions or areas where SME input is needed to address piracy and licensing issues in this system include:

* Clarifying the consistency model [S1](catalog.md), [S3](../runbooks/pearcare-fraud.md), [S4](../database/review-db.md), [S5](../database/catalog-db.md), [S8](../../sd/services/fulfillment.md)
* Determining the specific data stored in the cart database [S1](catalog.md), [S3](../runbooks/pearcare-fraud.md), [S4](../database/review-db.md), [S5](../database/catalog-db.md), [S8](../../sd/services/fulfillment.md)
* Determining the data stored in the account database [S1](catalog.md), [S3](../runbooks/pearcare-fraud.md), [S4](../database/review-db.md), [S5](../database/catalog-db.md), [S8](../../sd/services/fulfillment.md)
* Indexing strategies for the databases (further detail TBD) [S4](../database/payment-db.md)
* Backup strategies for the databases (further detail TBD) [S5](../database/review-db.md)
* Payment database durability and retention [S4](../database/payment-db.md), [S6](../../sd/services/review.md)
* The schema of the account database [S11](catalog-discovery.md)

Additionally, refining triage rules to better categorize and prioritize customer issues based on keywords could benefit from SME input [S3](../runbooks/pearcare-fraud.md). Improving payment processing by handling declined payments more efficiently and enhancing order fulfillment by automating refunds upon fulfillment failure are areas that require SME input.
