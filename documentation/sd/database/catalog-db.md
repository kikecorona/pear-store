# Catalog database

File: `data/catalog.db` (SQLite, WAL mode).

The catalog DB is the canonical store for every app available in the
storefront. It is read by every other service and written by exactly
two callers: the catalog service itself (on first-boot seeding) and the
review service (rating snapshot updates via the catalog API).

## Schema

```sql
CREATE TABLE apps (
    id            TEXT PRIMARY KEY,    -- stable, e.g. "app.001"
    name          TEXT NOT NULL,
    developer     TEXT NOT NULL,
    category      TEXT NOT NULL,
    price_cents   INTEGER NOT NULL,    -- 0 = free
    description   TEXT,
    icon_emoji    TEXT,
    rating        REAL    DEFAULT 0,   -- 1–5; pushed by review-svc
    rating_count  INTEGER DEFAULT 0,
    bundle_id     TEXT,                -- reverse DNS, e.g. "com.foo.bar"
    version       TEXT,                -- semver-ish
    size_mb       INTEGER
);

CREATE INDEX idx_apps_category ON apps(category);
```

## Seeding

On first boot, when `SELECT COUNT(*) FROM apps` returns 0, the service
imports every entry from `services/catalog/seed_data.py` and inserts
them in one batch. Subsequent restarts read from the existing rows;
editing the seed file does **not** retroactively update the database.
To re-seed, delete `data/catalog.db` and restart catalog-svc.

## Why ratings are denormalized here

A search query for "weather" needs to rank a couple thousand candidate
apps. Storing the latest aggregate (`rating`, `rating_count`) directly
on the `apps` row means search-svc does not need to join with
review-svc per result. Reviews and catalog agree on a one-way push:
review-svc updates catalog whenever a new review changes the average.

## Operational notes

* WAL is enabled (`PRAGMA journal_mode=WAL`) so search-svc reads do not
  block catalog-svc writes.
* `check_same_thread=False` is required because Flask reuses the
  connection across worker threads. All writes are short and
  autocommitted.
* No migrations framework is used. The schema is applied with
  `CREATE TABLE IF NOT EXISTS`; non-additive changes will need manual
  handling.

## Known gaps

* No `updated_at` column. We cannot tell when a row was last touched.
* Rating updates do not validate the caller — anyone who can reach the
  catalog API can clobber an app's average. In production this would be
  locked to review-svc.

## Access patterns

Typical query shapes for accessing apps in the catalog database include retrieving app information by ID, such as `SELECT * FROM catalog WHERE app_id = ?` [S3](cart-db.md). The most frequently accessed fields are likely to be `app_id`, `price_cents`, and other attributes related to product information [S2](cart-db.md).

The cart service stores information about items a user has added to their cart, including `user_id`, `items`, and other relevant data [S7](cart-db.md). However, the specific data stored in the cart database for each user is unclear [S4](cart-db.md) (further detail TBD).

When accessing apps in the catalog database, queries may also involve retrieving prices or product information by app ID, such as `SELECT price_cents FROM catalog WHERE app_id = ?` [S3](cart-db.md). The order service may also interact with the catalog service to retrieve prices or other relevant data [S8](../architecture/overview.md).

The cart service stores information about items a user has added to their cart, including `user_id`, `items`, and other relevant data [S7](cart-db.md). However, the specific data stored in the cart database for each user is unclear [S4](cart-db.md) (further detail TBD).

## Consistency / retention

To ensure data consistency and retention in the catalog database, consider the following strategies:

1. **Data persistence**: Orders are persisted in the `failed` state for audit purposes [S2](cart-db.md). This implies that data is retained for a certain period, although the exact duration is not explicitly stated [S1](cart-db.md).
2. **Schema design**: The cart database contains schemas for managing customer carts and orders, as well as retrieving product information from the catalog service [S2](cart-db.md). The schema likely includes attributes such as `user_id`, `items`, and `price_cents` [S7](cart-db.md).
3. **Service ownership**: The cart service owns the cart db database and is responsible for managing user carts, which are per-user, ephemeral, and idempotent [S4](cart-db.md).
4. **API endpoints**: Retrieving a user's cart items can be done using the `GET /carts/{user_id}` endpoint [S9](cart-db.md), while updating a user's cart can be achieved through the `POST /carts/{user_id}/clear` endpoint [S9](cart-db.md).

In terms of specific settings or configurations, there is no explicit information provided on how to ensure data consistency and retention. However, it is mentioned that disaster recovery procedures are not well-documented, and the system does not automatically refund orders when fulfillment fails [S6](cart-db.md). Further detail on ensuring data consistency and retention is TBD [further detail TBD].

Note: The catalog service provides product information, including prices, but its schema is not explicitly stated in the provided sources.

## Open Questions

The remaining open questions or areas that need SME input for the catalog database include:

1. Consistency model: What is the consistency model for the 'cart db' database, and how are data retention and backups handled? [S4](cart-db.md)
2. Specific data stored in the cart db: It is unclear what specific data is stored in the cart db for each user. [S6](cart-db.md)
3. Data stored in the account database: Further detail regarding the specific data stored in the account database for each user is TBD. [S3](cart-db.md)

These areas require SME input to clarify the consistency model, the specific data stored in the cart db, and the data stored in the account database.

Note that the sources do not provide explicit information on the following topics:

* The data consistency policy for the cart database (best guess: likely uses a consistent state model) [S4](cart-db.md)
* The schema of the cart database (attributes such as `user_id`, `items`, etc.) [S5](cart-db.md)

Further detail is TBD for these areas.
