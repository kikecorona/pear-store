# review service

The review server stores reveiws.

You POST to /reviews and it makes a review. The body has app_id, user_id, and stars (1 thru 5).
You can also include a `text` field for the body of the review.

After posting it talks to catelog so the average updates.

There is also a /reviews/<app_id> GET that gives all reviews for that app, but pagintation isnt
done yet. moderation is not handled.

## Overview

The review service's core duties include:

* Updating rating snapshots via the catalog API, which involves writing data to the catalog database [S1](../database/payment-db.md).
* Interacting with the catalog service to retrieve prices or other relevant data [S6](../database/pearcare-claim-db.md).

In terms of storage and processing functions, the review service is responsible for managing user-specific data, including cart information, in conjunction with the account database [S4](../database/pearcare-claim-db.md). The review service also plays a role in generating event logs related to rating snapshot updates via the catalog API [S2](catalog.md).

The review service's storage and processing functions are likely tied to the catalog database, which contains schemas for managing customer carts and orders, as well as retrieving product information from the catalog service [S5](../database/review-db.md). However, further detail on the specific storage and processing mechanisms used by the review service is TBD.

## Endpoints

The review service's available HTTP methods and paths are not explicitly described in the provided sources. However, based on the interactions between services mentioned in [S2](../pearcare/overview.md), we can infer some information about the review processing system.

The review service is responsible for owning ratings and free-text reviews, pushing aggregate ratings back into the catalog [S1](payment.md). This implies that there may be an endpoint or method within the review service's API for handling new reviews and updating aggregate ratings. 

One possible path for this endpoint is a POST request to initiate a review transaction. The specific data expected in the request body is not specified, but it likely involves a JSON object containing the rating and free-text review [S3](catalog.md).

The available HTTP methods and paths for the review service are:

* POST /apps/<app_id>/rating (used by review-svc) [S3](catalog.md)

Note that further detail regarding the specific data stored in the account database for each user is TBD [further detail TBD].

## Data model

The key entities in the review service's data model are:

* Users: represented by the `users` table in the account database, which stores user-specific data including cart information [S1](account.md).
* Carts: managed by the cart service, which owns the cart db database and is responsible for managing user carts, which are per-user, ephemeral, and idempotent [S3](../database/review-db.md).
* Orders: handled by the order service, which orchestrates interactions with the payment and fulfillment services.

The primary attributes of these entities are:

* Users:
	+ `user_id`: The ID of the user who owns the cart (integer) [S8].
	+ Cart information is not explicitly stated in the provided sources.
* Carts:
	+ `user_id`: The ID of the user who owns the cart (integer) [S8].
	+ `items`: A list of items in the cart, each containing attributes like `app_id`, `quantity`, and `price_cents` (array of objects) [S8].
* Orders:
	+ `id`: The unique ID of the order (integer) [S2](order.md).
	+ `user_id`: The ID of the user who placed the order (integer) [S2](order.md).
	+ `total_cents`: The total cost of the order (integer) [S2](order.md).
	+ `payment_id`: The payment ID associated with the order (string) [S2](order.md).

Note that further detail regarding the specific data stored in the account database for each user is TBD.

## Observability

The review service emits event logs related to interactions with the catalog service via the catalog API [S1](fulfillment.md). The catalog service generates system metrics and request traces [S2](catalog.md), which are likely related to cart management [S4](catalog.md), [S5](catalog.md).

Event logs from the catalog service can be used for monitoring interactions between the review service and the catalog service. System metrics from the catalog service can provide insights into the performance of the catalog service, such as CPU usage or memory consumption [further detail TBD] [S3](catalog.md). Request traces from the catalog service can help identify bottlenecks in cart management processes.

Note that the specific data stored in the cart database for each user is unclear (further detail TBD) [S4](catalog.md), but the Cart service stores information about items a user has added to their cart, including `user_id`, `items`, and other relevant data [S9].

## Open Questions

The review service has several open questions and unresolved issues that require SME input. One known gap is the handling of automatic refunds upon fulfillment failure, which requires human intervention [S2](catalog.md). Additionally, there is a pending question regarding what non-goals or out-of-scope work are explicitly excluded from this purchase data flow [S1](../runbooks/incident-response.md).

The system's architecture has been driven by modularity and decoupling of services, allowing for independent development and maintenance [S4]. However, the exact duration of data retention for orders in the `failed` state is not explicitly stated [S6], which may be an area where SME input is required.

Refining the triage rules to better categorize and prioritize customer issues based on keywords could also benefit from SME input [S3]. Furthermore, improving payment processing by handling declined payments more efficiently and enhancing order fulfillment by automating refunds upon fulfillment failure are areas that require SME input.
