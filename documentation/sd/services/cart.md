# Cart service

Per-user shopping cart. The cart is a flat list of `app_id` references —
no quantities, no variants, no bundles. Apps are unique within a cart
(adding the same app twice is a no-op).

## Endpoints

| Method | Path                                  | Notes |
| ------ | ------------------------------------- | ----- |
| GET    | `/carts/<user_id>`                    | Empty cart returns `{items: []}`. |
| POST   | `/carts/<user_id>/items`              | Body: `{app_id}`. Idempotent on duplicates. |
| DELETE | `/carts/<user_id>/items/<app_id>`     | 404 if item not present. |
| POST   | `/carts/<user_id>/clear`              | Called by order-svc after checkout. |

## Lifecycle

A cart is implicitly created the first time it is read or written. There
is no explicit create / delete. Carts are not durable across service
restarts.

## Notes

* The cart does not check that `app_id` exists in catalog. If a user
  sneaks in a bogus id, order-svc will catch it at checkout.
* No expiry / abandoned-cart sweep runs in this demo.

## Data model

The Cart service's data model consists of two primary entities: **cart** and **order**.

* The **cart** entity has the following attributes:
	+ `user_id`: The ID of the user who owns the cart (integer) [S3](../database/cart-db.md)
	+ `items`: A list of items in the cart, each containing attributes like `app_id`, `quantity`, and `price_cents` (array of objects) [S3](../database/cart-db.md)
* The **order** entity has the following attributes:
	+ `id`: The unique ID of the order (integer) [S3](../database/cart-db.md)
	+ `user_id`: The ID of the user who placed the order (integer) [S3](../database/cart-db.md)
	+ `total_cents`: The total cost of the order (integer) [S3](../database/cart-db.md)
	+ `payment_id`: The payment ID associated with the order (string) [S3](../database/cart-db.md)

The cart service owns the cart database and is responsible for managing user carts, which are per-user, ephemeral, and idempotent [S4](../database/catalog-db.md). Retrieving a user's cart items can be done using the `GET /carts/{user_id}` endpoint [S9](../database/review-db.md), while updating a user's cart can be achieved through the `POST /carts/{user_id}/clear` endpoint [S9](../database/review-db.md).

The data consistency policy for the cart database is likely to use a consistent state model, as implied by the account database following a state machine for "pending → paid → fulfilled" transactions [S10](../database/pearcare-claim-db.md). The exact duration of data retention is not explicitly stated [S1](../database/cart-db.md), but it is retained for audit purposes [S2](../database/catalog-db.md).

Note: Further detail regarding the specific data stored in the cart db for each user and the account database is TBD [S8](../database/catalog-db.md).

## Observability

The Cart service stores information about items a user has added to their cart, including `user_id`, `items`, and other relevant data [S9](../database/catalog-db.md), [S10](../database/catalog-db.md). The specific data stored in the cart database for each user is unclear (further detail TBD) [S4](../database/cart-db.md).

To retrieve a user's cart items, the `GET /carts/{user_id}` endpoint can be used [S2](../database/pearcare-claim-db.md), [S3](../database/catalog-db.md). To update a user's cart, the `POST /carts/{user_id}/clear` endpoint can be used [S2](../database/pearcare-claim-db.md), [S3](../database/catalog-db.md).

The Cart service owns the cart db database and is responsible for managing user carts, which are per-user, ephemeral, and idempotent [S1](../database/cart-db.md), [S2](../database/pearcare-claim-db.md), [S3](../database/catalog-db.md).

## Open Questions

Outstanding questions or areas requiring SME input for the Cart service include:

* Clarifying the consistency model for the 'cart db' database, as it is not explicitly stated [S5](../database/cart-db.md). However, it can be inferred that the cart database likely uses a consistent state model to manage its data.
* Determining the specific data stored in the account database for each user, including further detail regarding the schema of the account database [S1](../database/review-db.md), [S3](../database/payment-db.md).
* Improving observability features, such as metrics, traces, or structured logs, to better understand and troubleshoot issues with the Cart service [S2](../runbooks/incident-response.md).
* Handling automatic refunds upon fulfillment failure, which is currently not handled automatically and requires human intervention [S2](../runbooks/incident-response.md).
* Refining triage rules to better categorize and prioritize customer issues based on keywords [S3](../database/payment-db.md).
* Improving payment processing by handling declined payments more efficiently.
* Enhancing order fulfillment by automating refunds upon fulfillment failure.

Further detail regarding the specific data stored in the account database for each user is TBD [S7](../database/pearcare-claim-db.md).
