# account database

`data/account.db`. There is a users table.

It has the user info in it.

Two demo users get inserted at startup.

## Overview

The account database stores user-specific data, including cart information [S1](../architecture/overview.md). The order service owns the account database, which is responsible for managing the state machine for "pending → paid → fulfilled" transactions [S1](../architecture/overview.md). This includes orchestrating interactions with the payment and fulfillment services.

The account database does not store real-time authentication or authorization data, as Pear Store uses module-level dictionaries for each service, which are lost on restart [S2](../architecture/overview.md). The system also lacks observability features such as metrics, traces, or structured logs [S2](../architecture/overview.md).

The Fulfillment Service (FUL) processes replacement orders and sends entitlements in response to POST requests from the CL service [S3](../architecture/data-flow-purchase.md). However, further detail regarding the specific data stored in the account database for each user is TBD.

## Schema

<!-- SME-PLACEHOLDER:q-sd-d012b03758 START -->
> ⏳ **Waiting for SME** — *Topic:* Schema
>
> *Question:* Describe the structure of the 'users table' in the account database.
> *Best guess (low-confidence):* The 'users table' in the account database does not appear to be described in the provided sources [S1] and [S2]. However, based on [S1], we can infer that the order-svc (orchestrator) interacts with various services, including cart-svc, catalog-svc, payment-svc, and fulfillment-svc. The users table might be related to the user's information stored in the cookie, which holds `{id, email, display_name}` [S1].

A possible structure of the 'users table' could include columns for:

* `id` (primary key) [S1]
* `email`
* `display_name`

However, without explicit mention of the users table in the provided sources, this is a partial answer with further detail TBD.
> *Asked:* on 2026-06-27 · *Status:* pending · *Question ID:* `q-sd-d012b03758`
<!-- SME-PLACEHOLDER:q-sd-d012b03758 END -->
