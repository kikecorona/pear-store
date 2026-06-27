# pearcare claim db

claims go in pearcare-claim.db. one big table. status and resolution and stuff.

replacement column has json.

## Overview

The PearCare claim database is administered by the **PearCare Claim** service, which owns the claim lifecycle (`filed → triaged → approved | denied`) [S1](../architecture/overview.md). When a claim resolves to a replacement, the fulfillment service reuses its logic from PearCare's claim service [S2](cart-db.md). The data flow diagram shows that for replacement resolutions, the CL service sends a request to the Fulfillment Service (FUL) with the order ID and claim ID [S3](../architecture/data-flow-purchase.md), [S4](../architecture/data-flow-pearcare-claim.md).

Therefore, the database management or claims processing service responsible for administering the pearcare claim database is the **Fulfillment Service**.

## Schema

The table in pearcare-claim.db is likely a payment table, which includes the following fields and their types:

* `id`: The unique ID of the payment (integer) [S5](payment-db.md)
* `user_id`: The ID of the user who made the payment (integer) [S4](cart-db.md), [S5](payment-db.md)
* `items`: A list of items in the payment, each containing attributes like `app_id`, `quantity`, and `price_cents` (array of objects) [S4](cart-db.md)
* `price_cents`: The total cost of the payment in cents (integer) [S4](cart-db.md), [S5](payment-db.md)
* `status`: The status of the payment (e.g. active, abandoned) (string) [S5](payment-db.md)

Note that there is no explicit discussion of an "order" table in the provided sources, but it is mentioned as a separate service in [S4](cart-db.md). Further detail on the structure of the order table TBD.

## Access patterns

The account database stores data accessed by various services. Specifically, the order service owns the account database and is responsible for managing user-specific data, including cart information [S1](../architecture/overview.md). The Fulfillment Service (FUL) also interacts with the account database to process replacement orders [S3](../architecture/data-flow-purchase.md).

The typical query shapes on the pearcare-claim-db are not explicitly mentioned in the provided sources. However, based on the information about the account database and its interactions with other services, it can be inferred that queries may involve retrieving user-specific data, such as cart information or order status [S1](../architecture/overview.md).

The catalog service also interacts with the account database to retrieve prices or other relevant data for orders [S4](catalog-db.md). The schema of the account database is not explicitly described in the provided sources, but it likely includes attributes such as `user_id`, `items`, and `price_cents` [S5](payment-db.md).

Further detail regarding the specific data stored in the account database for each user is TBD.

## Consistency / retention

Data consistency and retention in pearcare-claim-db are ensured through various measures. Data retention is handled by persisting orders in the `failed` state for audit purposes, although the exact duration of data storage is not explicitly stated [S1](payment-db.md). The cart database contains schemas for managing customer carts and orders, as well as retrieving product information from the catalog service [S2](cart-db.md), which likely includes attributes such as `user_id`, `items`, and `price_cents` [S5](catalog-db.md).

The account database stores user-specific data, including cart information [S1](payment-db.md), and follows a state machine for "pending → paid → fulfilled" transactions [S1](payment-db.md). This implies that the cart database likely uses a consistent state model to manage its data [S2](cart-db.md). The Fulfillment Service (FUL) processes replacement orders and sends entitlements in response to POST requests from the CL service [S3](account-db.md), suggesting that the fulfillment service plays a key role in managing entitlements and replacements.

The cart service owns the cart db database and is responsible for managing user carts, which are per-user, ephemeral, and idempotent [S4](cart-db.md). Retrieving a user's cart items can be done using the `GET /carts/{user_id}` endpoint [S9], while updating a user's cart can be achieved through the `POST /carts/{user_id}/clear` endpoint [S9].

Note: Further detail regarding the specific data stored in the account database for each user is TBD [S3](account-db.md).

## Open Questions

There are open questions and areas of uncertainty regarding the 'cart db' database. The cart information is stored in the account database [S1](cart-db.md), but further detail regarding the specific data stored in the account database for each user is TBD [S3]. Additionally, the system lacks observability features such as metrics, traces, or structured logs [S2].

Note that this uncertainty also implies that there may be similar issues with pearcare-claim-db, given its similarity to cart db. However, further investigation would be required to confirm this.
