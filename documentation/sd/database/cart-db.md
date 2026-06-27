# cart db

`data/cart.db`. Cart items.

## Overview

The 'cart db' database is owned by the **cart** service [S2](../architecture/overview.md). The primary function of the cart service is to manage user carts, which are per-user, ephemeral, and idempotent [S1](../architecture/overview.md), [S2](../architecture/overview.md). This means that the cart service stores information about the items a user has added to their cart, but this state does not survive a restart. 

The cart service interacts with other services during checkout, including **order**, **payment**, and **fulfillment** [S3](../architecture/data-flow-purchase.md). Specifically, when a user initiates a checkout, the order service retrieves the cart contents from the cart service [S3](../architecture/data-flow-purchase.md). The fulfillment service is then responsible for minting licenses and per-platform download URLs [S2](../architecture/overview.md).

When a claim resolves to a replacement, the fulfillment service reuses its logic from PearCare's claim service [S2](../architecture/overview.md). This suggests that the fulfillment service plays a key role in managing entitlements and replacements.

## Schema

The 'cart database' contains the following schemas, datasets, or attributes with their respective data types:

* **cart**: This is a service that manages customer carts. The cart schema likely includes attributes such as:
	+ `user_id`: The ID of the user who owns the cart (integer)
	+ `items`: A list of items in the cart, each containing attributes like `app_id`, `quantity`, and `price_cents` (array of objects)
* **order**: This service handles order creation and management. The order schema likely includes attributes such as:
	+ `id`: The unique ID of the order (integer)
	+ `user_id`: The ID of the user who placed the order (integer)
	+ `total_cents`: The total cost of the order (integer)
	+ `payment_id`: The payment ID associated with the order (string)
* **catalog**: This service provides product information, including prices. The catalog schema likely includes attributes such as:
	+ `app_id`: The ID of the app being sold (integer)
	+ `price_cents`: The price of the app in cents (integer)

These schemas are used to manage customer carts and orders, and to retrieve product information from the catalog service.

Note: The data types mentioned above are based on the provided sources and may not be exhaustive. Further detail TBD.

## Access patterns

Entities that access, retrieve, or update data in the 'cart database', 'shopping cart database', or 'ecommerce database' include:

* The order-svc (orchestrator) [S2](../architecture/data-flow-purchase.md), which interacts with various services including cart-svc, catalog-svc, payment-svc, and fulfillment-svc.
* The frontend (FE) [S3](../architecture/data-flow-purchase.md), which sends requests to the order-svc for checkout.

Common query patterns or structures in the 'cart database' include:

* Retrieving a user's cart items: `GET /carts/{user_id}` [S2](../architecture/data-flow-purchase.md)
* Updating a user's cart: `POST /carts/{user_id}/clear` [S3](../architecture/data-flow-purchase.md)

Note that the sources do not provide explicit information on the structure of the 'users table' in the account database, so further detail is TBD.

## Consistency / retention

For the purpose of this POC, retention is permanent.


## Open Questions

There are open questions and areas of uncertainty regarding the 'cart db' database. The cart information is stored in the account database [S1](../architecture/overview.md), but further detail regarding the specific data stored in the account database for each user is TBD [S3](../architecture/data-flow-purchase.md). Additionally, the system lacks observability features such as metrics, traces, or structured logs [S2](../architecture/overview.md).

The cart db is per-user and ephemeral, idempotent, and survives across navigation but not restarts [S2](../architecture/overview.md). However, it is unclear what specific data is stored in the cart db for each user.

It is also worth noting that the system has a known gap regarding automatic refunds when fulfillment fails [S5](domain=sd, source=sd/architecture/data-flow-purchase.md).
