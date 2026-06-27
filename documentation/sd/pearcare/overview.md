# PearCare overview

PearCare is the warranty product that sits next to the Pear Store. It
lets a user buy a protection plan against an app they own, and lets
them file claims when something goes wrong. PearCare is its own
sub-system in the codebase — own services, own ports, own data — but
it borrows three things from the storefront:

| What                | From which service       | Why |
| ------------------- | ------------------------ | --- |
| App metadata        | catalog-svc              | so a plan page can show the app it covers |
| User identity       | account-svc              | enrollments and claims are keyed by `user_id` |
| Replacement licenses | fulfillment-svc          | when a claim resolves to a replacement |

## Services

* **`pearcare-plan`** (port 15101) — the catalog of plans, plus
  per-user enrollments. See `plan-service.md`.
* **`pearcare-claim`** (port 15102) — claim filing, triage, approval,
  and the hooks that resolve a claim into a real-world action. See
  `claim-service.md`.

## Plan tiers

| Tier             | Coverage |
| ---------------- | -------- |
| `pearcare`       | Accidental damage, support |
| `pearcare_plus`  | + priority support, unlimited claims, data recovery |
| `pearcare_loss`  | + loss / theft replacement |

A plan is offered for a specific subset of apps — see `APP_PLAN_MAP`
in `plan/seed_data.py`. Most apps in the storefront have **no** plan;
only premium creative, pro dev, and a handful of premium games do.

## End-to-end story

1. User opens an app detail page. The frontend calls
   `pearcare-plan /plans/for-app/<id>`. If plans exist, a radio group
   of plans is rendered next to the buy button.
2. User picks a plan and adds the app to their cart. The plan choice
   is held in the session.
3. User checks out. The order service runs a normal storefront
   purchase. Once the order is in `fulfilled` state, the frontend
   calls `pearcare-plan /enrollments` for each pending plan, which
   creates an enrollment dated from now.
4. Later, the user files a claim from their account page. The frontend
   calls `pearcare-claim /claims`. The claim is auto-triaged, and from
   there an operator (or in this demo, a follow-up call) approves the
   claim. The hook fires and the user either gets a vendor ticket or a
   replacement license.

## Why a separate sub-system

The storefront cares about catalog, money, and downloads. PearCare
cares about enrollments, time windows, and claim eligibility. Coupling
them tightly would mean the storefront's order-svc state machine had
to grow `enrolled` and `claim-pending` states it does not need. A
separate sub-system also means PearCare can be deployed and rolled back
independently, which is important the first few months a warranty
product is live.

## Service map

**Constituent Services of PearCare**

PearCare consists of several constituent services, including:

* **Cart**: a per-user, ephemeral service that stores state across navigation but not restarts [S1](../architecture/overview.md).
* **Order**: an orchestrator for checkout that talks to **Payment** and **Fulfillment**, managing the "pending → paid → fulfilled" lifecycle [S1](../architecture/overview.md).
* **Payment**: a service that wraps three provider hooks (Stripe, PayPal, PearCard) and is chosen by token prefix [S1](../architecture/overview.md).
* **Fulfillment**: a service that mints licenses and per-platform download URLs, reused from the claim service for replacement entitlements [S2](../database/cart-db.md), [S5](../architecture/overview.md).
* **Review**: a service that owns ratings and free-text reviews, pushing aggregate ratings back into the catalog [S1](../architecture/overview.md).

**Inter-Service Communication Architecture**

The inter-service communication architecture of PearCare involves several key interactions:

* The frontend initiates a claim, which triggers a POST request to the **PearCare Claim** (CL) service from the frontend (FE) [S3](claim-service.md).
* The CL service determines the resolution type and triggers the corresponding hook: repair resolutions dispatch a request to the Repair Vendor Hook (RV), while replacement resolutions send a POST request to the Fulfillment Service (FUL) with the order ID and claim ID [S3](claim-service.md), [S4](../architecture/data-flow-purchase.md).
* The CL service sends a request to the FUL service for replacement entitlements, which processes replacement orders and sends entitlements in response [S3](claim-service.md), [S4](../architecture/data-flow-purchase.md).

Note that further detail regarding the specific data stored in the account database for each user is TBD [S7](../database/account-db.md).

## Ownership boundaries

The PearCare system consists of several services that own specific data and responsibilities.

* **PearCare Plan** owns warranty plan definitions and per-user enrollments [S2](../architecture/overview.md). It does not store anything about claims.
* **PearCare Claim** owns the claim lifecycle (`filed → triaged → approved | denied`) [S3](../database/pearcare-claim-db.md), [S12](claim-service.md). Approval fans out to either a vendor-repair hook or the main fulfillment service for replacement entitlements [S2](../architecture/overview.md).
* The account database is owned by the Order Service and stores user-specific data, including cart information [S1](../database/review-db.md), [S5](../database/pearcare-claim-db.md). The Fulfillment Service (FUL) also interacts with the account database to process replacement orders [S3](../database/pearcare-claim-db.md), [S5](../database/pearcare-claim-db.md).

The services are split as follows:

* PearCare Claim (CL): handles claim filing and resolution determination
* PearCare Plan (PL): provides enrollment information
* Repair Vendor Hook (RV): dispatches repair requests to vendors
* Fulfillment Service (FUL): processes replacement orders

When a claim resolves to a replacement, the fulfillment service reuses its logic from PearCare's claim service [S2](../architecture/overview.md), [S9](../database/cart-db.md). The CL service sends a POST request to the FUL service with the order ID and claim ID for replacement resolutions [S4](../architecture/data-flow-purchase.md).

The data flow diagram shows that for replacement resolutions, the CL service sends a request to the Fulfillment Service (FUL) with the order ID and claim ID [S3](../database/pearcare-claim-db.md), [S4](../architecture/data-flow-purchase.md). The triage rules specify that real eligibility logic should live here, but the default rules are intentionally simple [S1](../database/review-db.md), [S4](../architecture/data-flow-purchase.md).

The key endpoints involved in this process are:

* **PearCare Claim** (`CL`): owns the claim lifecycle and determines the resolution type.
* **Fulfillment Service** (`FUL`): handles replacement entitlements.
* **Repair Vendor Hook** (`RV`): handles repair resolutions.

Note that further detail regarding the specific data stored in the account database for each user is TBD [further detail TBD].

## Why this shape

The development of a standalone subsystem for PearCare management was driven by the need to modularize and decouple services, allowing for independent development and maintenance. This design choice resulted in the creation of separate services for claim filing and resolution determination (PearCare Claim), enrollment information (PearCare Plan), repair requests (Repair Vendor Hook), and replacement orders (Fulfillment Service) [S1](../architecture/data-flow-purchase.md).

The key factors driving this design decision were:

* The need to reuse fulfillment logic from PearCare's claim service, as mentioned in [S2](../architecture/overview.md) and [S5](../database/pearcare-claim-db.md).
* The requirement for a modular, service-oriented architecture, where each service has a specific responsibility and can be developed and maintained independently.
* The desire to push ratings updates instead of pulling them, as described in [S2](../architecture/overview.md).

The standalone subsystem for PearCare management was designed to handle the claim lifecycle, including filing, triage, approval/denial, and fulfillment. The key endpoints involved in this process are:

* **PearCare Claim** (`CL`): owns the claim lifecycle and determines the resolution type.
* **Fulfillment Service** (`FUL`): handles replacement entitlements.
* **Repair Vendor Hook** (`RV`): handles repair resolutions.

The design allows for a clear separation of concerns, enabling each service to be developed and maintained independently. The use of a modular architecture also facilitates scalability and flexibility in managing PearCare claims [S1](../architecture/data-flow-purchase.md).

(further detail TBD)

## What is not in this system

The PearCare system intentionally excludes storing anything about claims in its **PearCare plan**. The claim lifecycle (`filed → triaged → approved | denied`) is owned by the **PearCare claim** service, which determines the resolution type and triggers the corresponding hook [S1](../architecture/overview.md). For repair resolutions, the **PearCare claim** service dispatches a request to the Repair Vendor Hook (RV) [S3](../architecture/data-flow-purchase.md), while for replacement resolutions, it sends a POST request to the Fulfillment Service (FUL) with the order ID and claim ID [S4](../database/pearcare-claim-db.md).

The PearCare system also intentionally keeps its triage rules simple, as specified in the `claim/app.py` file [S3](../architecture/data-flow-purchase.md). The exact duration of data retention for orders in the `failed` state is not explicitly stated [S6](integration.md), but it is handled by persisting orders in the `failed` state for audit purposes.

The Fulfillment Service (FUL) reuses its logic from PearCare's claim service when a claim resolves to a replacement, suggesting that the fulfillment service plays a key role in managing entitlements and replacements [S7](../database/cart-db.md).
