# payment db

`data/payment.db`. There's a payments table somewhere in here.

Has fields like id and amount and provider. Refunds set a refund_id column.

We dont keep history of state changes, just the latest status.

## Overview

The payment database is owned by the **payment** service [S1](cart-db.md). However, it is unclear what specific data the payment database stores, as there is no explicit mention of its schema or contents in the provided sources (further detail TBD).

The cart service interacts with the payment service during checkout [S3](catalog-db.md), and the order service may also interact with the payment service to retrieve relevant data [S8](../architecture/overview.md). The fulfillment service is responsible for minting licenses and per-platform download URLs, but it does not appear to be directly involved in storing or managing payment information.

It is worth noting that the cart database contains schemas for managing customer carts and orders, as well as retrieving product information from the catalog service [S2](cart-db.md). However, this does not necessarily imply that the cart database stores payment information.

## Schema

**Payment Database Tables**

The payment database contains tables related to managing payments and transactions.

### Payment Table Structure

The payment table likely includes columns for:

* `id` (primary key) [S1](cart-db.md)
* `user_id`
* `order_id`
* `payment_method` (e.g. stripe, paypal, pearcard)
* `amount_cents`
* `status` (e.g. pending, paid, fulfilled)

### Order Table Structure

The order table likely includes columns for:

* `id` (primary key) [S1](cart-db.md)
* `user_id`
* `cart_id`
* `payment_id`
* `status` (e.g. pending, paid, fulfilled)
* `fulfillment_status`

### Cart Table Structure

The cart table likely includes columns for:

* `id` (primary key) [S7](catalog-db.md)
* `user_id`
* `items` (list of item IDs or objects)
* `price_cents`
* `status` (e.g. active, abandoned)

Note: The exact structure and field types may vary depending on the specific requirements of the payment database.

References:

[S1](cart-db.md) (domain=sd, source=sd/database/cart-db.md)
[S2](account-db.md) (domain=sd, source=sd/database/account-db.md)
[S7](catalog-db.md) (domain=sd, source=sd/database/catalog-db.md)

## Access patterns

**Payment Database Access**

The payment database is accessed by the **order** service, which interacts with various services including **payment**, during checkout. The order service retrieves the cart contents from the cart service and then uses this information to interact with the payment service [S4](cart-db.md).

**Typical Query Shapes**

Typical query shapes for accessing apps in the catalog database include retrieving app information by ID, such as `SELECT * FROM catalog WHERE app_id = ?` [S3](catalog-db.md). The most frequently accessed fields are likely to be `app_id`, `price_cents`, and other attributes related to product information [S2](../architecture/overview.md).

**Payment Database Writers**

The payment database is written by the **payment** service, which wraps three provider hooks (`stripe`, `paypal`, `pearcard`) [S2](../architecture/overview.md). The order service also interacts with the payment service during checkout.

**Catalog Service Interactions**

The catalog service is read by every other service and written by exactly two callers: the catalog service itself (on first-boot seeding) and the review service (rating snapshot updates via the catalog API) [S5](catalog-db.md). The order service may also interact with the catalog service to retrieve prices or other relevant data [S6](catalog-db.md).

**Cart Service Interactions**

The cart service interacts with other services during checkout, including **order**, **payment**, and **fulfillment** [S4](cart-db.md). Specifically, when a user initiates a checkout, the order service retrieves the cart contents from the cart service.

## Consistency / retention

For the purpose of this POC retention is permanent.


## Open Questions

The open questions related to the payment database that need SME input are:

1. Consistency model: What is the consistency model for the 'cart db' database, and how are data retention and backups handled? [S2](cart-db.md), [S5](catalog-db.md)
2. Specific data stored in the cart db: It is unclear what specific data is stored in the cart db for each user. [S3](cart-db.md), [S5](catalog-db.md)
3. Data stored in the account database: Further detail regarding the specific data stored in the account database for each user is TBD. [S1](catalog-db.md), [S3](cart-db.md)

Note that the consistency model for the 'cart db' database and how data retention and backups are handled is unclear, as it can be inferred but not explicitly stated [S2](cart-db.md). The cart database likely uses a consistent state model to manage its data, but this is only a best guess with low confidence.
