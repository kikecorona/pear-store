# review db

reviews go in here

`data/review.db` has a reviews table with the reveiw rows. We push the average to
catelog after each insert.

There is an index on app_id i think.

## Overview

The account database is owned by the Order Service and stores user-specific data, including cart information [S1](pearcare-claim-db.md). The Fulfillment Service (FUL) also interacts with the account database to process replacement orders [S3](pearcare-claim-db.md). However, further detail regarding the specific data stored in the account database for each user is TBD [further detail TBD].

The payment database, on the other hand, is accessed by the Order Service and stores information related to payments during checkout [S4](payment-db.md).

## Schema

The database contains several tables, collections, and fields with specific types.

**Payment Table**

The payment table is likely a payment table that includes the following fields:

* `id` (primary key): [S7]
* `user_id`
* `items`: A list of item IDs or objects [S2](payment-db.md)
* `price_cents`: The price in cents, an integer value [S6](cart-db.md)
* `status`: The status of the payment, e.g. active or abandoned, a string value [S2](payment-db.md)

**Cart Table**

The cart table likely includes columns for:

* `user_id`: The ID of the user who owns the cart, an integer value [S6](cart-db.md)
* `items`: A list of items in the cart, each containing attributes like `app_id`, `quantity`, and `price_cents` (array of objects) [S6](cart-db.md)

**Order Table**

The order table likely includes columns for:

* `id`: The unique ID of the order, an integer value [S4](payment-db.md)
* `user_id`: The ID of the user who placed the order, an integer value [S4](payment-db.md)
* `total_cents`: The total cost of the order, an integer value [S4](payment-db.md)
* `payment_id`: The payment ID associated with the order, a string value [S4](payment-db.md)

**Catalog Table**

The catalog table likely includes columns for:

* `app_id`: The ID of the app being sold, an integer value [S6](cart-db.md)
* `price_cents`: The price of the app in cents, an integer value [S6](cart-db.md)

## Access patterns

Entities that access, retrieve, or update data in the 'cart database' include the cart service [S3](cart-db.md). The payment database writers also interact with this database [S4](payment-db.md).

Common query patterns or structures in the 'cart database' include retrieving a user's cart by `user_id` and accessing the list of items (`items`) within it [S1](cart-db.md). Other common queries may involve updating the price_cents field for an item in the cart, although the exact structure and field types may vary depending on the specific requirements of the payment database [S8](payment-db.md).

The fields that are likely used as hot keys in the 'cart database' include `user_id` and `items`, which are used to uniquely identify a user's cart and its contents, respectively [S9](cart-db.md). The `id` field is also a primary key in this database [S2](payment-db.md), but it may not be used as frequently as other fields.

Further detail on the query shapes and hot keys is TBD.

## Consistency / retention

Based on the provided sources, here are the durability, retention, index, and backup strategies for the databases:

**Catalog Database**

* Data persistence: Orders are persisted in the `failed` state for audit purposes [S1](cart-db.md), implying data retention for a certain period (although the exact duration is not explicitly stated) [S2](payment-db.md).
* Schema design: The catalog database contains schemas for managing customer carts and orders, as well as retrieving product information from the catalog service [S2](cart-db.md). The schema likely includes attributes such as `user_id`, `items`, and `price_cents` [S5](catalog-db.md).
* Service ownership: The catalog service owns the catalog database and is responsible for managing user carts, which are per-user, ephemeral, and idempotent [S4](cart-db.md).

**Cart Database**

* Data consistency policy: The data consistency policy for the cart database is not explicitly stated in the provided sources. However, it can be inferred that the account database, which stores user-specific data including cart information [S1](../architecture/overview.md), follows a state machine for "pending → paid → fulfilled" transactions [S1](../architecture/overview.md). This implies that the cart database likely uses a consistent state model to manage its data [S3](cart-db.md).
* Data retention: Orders are persisted in the `failed` state for audit purposes [S2](cart-db.md), implying data retention for a certain period (although the exact duration is not explicitly stated) [S1](payment-db.md).

**Payment Database**

* Data consistency policy: The payment database follows a state machine for "pending → paid → fulfilled" transactions [S1](../architecture/overview.md).
* Data retention: Orders are persisted in the `failed` state for audit purposes [S2](cart-db.md), implying data retention for a certain period (although the exact duration is not explicitly stated) [S1](payment-db.md).

**Index and Backup Strategies**

* Indexing: The sources do not provide information on indexing strategies for the databases.
* Backup strategies: The sources do not provide information on backup strategies for the databases.

Note that further detail regarding specific data stored in the account database for each user is TBD [S8](account-db.md).

## Open Questions

Based on the provided sources, the following questions or concerns need SME input regarding this database:

1. **Consistency model for the 'cart db' database**: The consistency model is not explicitly stated in the provided sources [S6](cart-db.md). However, it can be inferred that the account database follows a state machine for "pending → paid → fulfilled" transactions [S1](catalog-db.md), which implies that the cart database likely uses a consistent state model to manage its data.
2. **Specific data stored in the cart db**: It is unclear what specific data is stored in the cart db for each user [S3](payment-db.md), [S5](pearcare-claim-db.md).
3. **Data stored in the account database**: Further detail regarding the specific data stored in the account database for each user is TBD [S1](catalog-db.md), [S3](payment-db.md).
4. **Data retention and backups for the 'cart db' database**: The exact duration of data storage is not explicitly stated [S1](catalog-db.md), and it is unclear how data retention and backups are handled.
5. **Observability features**: The system lacks observability features such as metrics, traces, or structured logs [S2](cart-db.md).
6. **Payment database durability and retention**: How does the payment database ensure data durability and retention, including any indexing or backup strategies? [S3](payment-db.md)
7. **Schema of the account database**: The schema of the account database is not explicitly described in the provided sources [S11](pearcare-claim-db.md).

These questions and concerns highlight areas where SME input is necessary to clarify the consistency model, specific data stored in the cart db and account database, data retention and backups, observability features, payment database durability and retention, and the schema of the account database.
