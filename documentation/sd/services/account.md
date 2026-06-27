# account

Handles user logins. Has a register endpoint and a login endpoint.

There are two demo users seeded at startup, ada and grace.

Passwords are stored as plain text for now, will fix later.

## Overview

The account service is responsible for managing user-specific data, including cart information [S1](../database/account-db.md). It stores this data in the account database, which is owned by the order service and manages the state machine for "pending → paid → fulfilled" transactions [S1](../database/account-db.md). The account service does not store real-time authentication or authorization data, as Pear Store uses module-level dictionaries for each service, which are lost on restart [S2](../pearcare/overview.md).

The key responsibilities of the account service include:

* Managing user-specific data, including cart information
* Storing this data in the account database
* Participating in the state machine for "pending → paid → fulfilled" transactions

Further detail regarding the specific data stored in the account database for each user is TBD [S7](../database/payment-db.md).

## Endpoints

<!-- SME-PLACEHOLDER:q-sd-790a964e00 START -->
> ⏳ **Waiting for SME** — *Topic:* Endpoints
>
> *Question:* What are the register and login endpoints' methods, paths, and handlers in the account service?
> *Best guess (low-confidence):* The registration endpoint's HTTP method and URL are not explicitly mentioned in the provided sources. However, based on the information about inter-service communication architecture, it can be inferred that the frontend initiates a claim with a POST request to the **PearCare Claim** (CL) service [S3].

The authentication endpoint's HTTP method and URL are also not explicitly mentioned. The sources do mention that the cart service owns the cart db database and is responsible for managing user carts, which are per-user, ephemeral, and idempotent [S4]. However, this information does not provide details about the registration or authentication endpoints.

The order service's HTTP method and URL can be inferred from its function as an orchestrator for checkout that talks to **Payment** and **Fulfillment**, managing the "pending → paid → fulfilled" lifecycle [S1].

The fulfillment service's HTTP method is POST, as it mints licenses and per-platform download URLs in response to a request from the CL service [S3], [S4]. The URL for this endpoint is not explicitly mentioned.

The payment service's HTTP method and URL are also not explicitly mentioned. However, based on its function of wrapping three provider hooks (Stripe, PayPal, PearCard) and being chosen by token prefix [S1], it can be inferred that the payment service interacts with external providers through specific endpoints.

In summary:

* Registration endpoint: Not explicitly mentioned
* Authentication endpoint: Not explicitly mentioned
* Order service: Orchestrates checkout, interacting with Payment and Fulfillment services; HTTP method and URL TBD
* Fulfillment service: POST request to mint licenses and per-platform download URLs; URL TBD
* Payment service: Wraps three provider hooks (Stripe, PayPal, PearCard); HTTP method and URL TBD
> *Asked:* on 2026-06-27 · *Status:* pending · *Question ID:* `q-sd-790a964e00`
<!-- SME-PLACEHOLDER:q-sd-790a964e00 END -->

## Data model

The key entities in the account service are:

* Users: represented by the `users` table in the account database, which stores user-specific data including cart information [S1](../architecture/overview.md).
* Carts: managed by the cart service, which owns the cart db database and is responsible for managing user carts, which are per-user, ephemeral, and idempotent [S3](../database/payment-db.md).
* Orders: handled by the order service, which orchestrates interactions with the payment and fulfillment services.

The primary attributes of these entities are:

* Users:
	+ `user_id`: The ID of the user who owns the cart (integer) [S8](../database/cart-db.md).
	+ Cart information (further detail TBD).
* Carts:
	+ `user_id`: The ID of the user who owns the cart (integer) [S8](../database/cart-db.md).
	+ `items`: A list of items in the cart, each containing attributes like `app_id`, `quantity`, and `price_cents` (array of objects) [S8](../database/cart-db.md).
* Orders:
	+ `id`: The unique ID of the order (integer) [S8](../database/cart-db.md).
	+ `user_id`: The ID of the user who placed the order (integer) [S8](../database/cart-db.md).
	+ `total_cents`: The total cost of the order (integer) [S8](../database/cart-db.md).
	+ `payment_id`: The payment ID associated with the order (string) [S8](../database/cart-db.md).

Note that further detail regarding the specific data stored in the account database for each user is TBD.

## Observability

The account service generates audit records, performance indicators, and distributed request data for monitoring and troubleshooting purposes. Specifically:

Audit records are generated when orders are persisted in the `failed` state for a certain period, although the exact duration is not explicitly stated [S1](../database/review-db.md) [S2](../database/pearcare-claim-db.md). These records imply data retention for auditing purposes.

Performance indicators may include metrics related to user cart management, such as the number of carts created or updated by the catalog service [S4].

Distributed request data may be generated when the account service interacts with other services, including the catalog service, which is responsible for managing user carts and retrieving product information [S3](../database/payment-db.md) [S5]. The catalog database schema likely includes attributes such as `user_id`, `items`, and `price_cents` [S2](../database/pearcare-claim-db.md) [S5].

Further detail on specific performance indicators and distributed request data generated by the account service is TBD.

## Open Questions

Outstanding concerns or unresolved matters related to the account management service necessitating subject matter expert input include:

* Clarifying the consistency model used by the cart db and the account database, as well as the specific data stored in these databases for each user [S2](../database/catalog-db.md), [S5](../database/account-db.md).
* Resolving the issue of automatic refunds not being handled upon fulfillment failure [S3](../runbooks/incident-response.md).
* Determining the exact duration of data retention for orders in the failed state [S6](../database/pearcare-claim-db.md).
* Improving payment processing by handling declined payments more efficiently.
* Enhancing order fulfillment by automating refunds upon fulfillment failure.
* Refining the triage rules to better categorize and prioritize customer issues based on keywords [S3](../runbooks/incident-response.md).
* Understanding the data stored in the account database, including cart information and user-specific data [S7](../database/review-db.md), [S8](../pearcare/integration.md).

Further detail is needed regarding the specific data stored in the account database for each user [S12].
