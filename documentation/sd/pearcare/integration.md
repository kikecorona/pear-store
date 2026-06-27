# PearCare integration with the storefront

PearCare lives next to the storefront, not inside it. There are exactly
three integration points; each one is small and traceable.

## 1. App detail page — show plans

When the storefront frontend renders an app detail page it calls:

```
GET pearcare-plan /plans/for-app/<app_id>
```

If the response is non-empty, a radio group is rendered next to the
"Add to cart" button, with a default of "No protection". The selection
is held in the Flask session under `pending_plans`, not pushed to the
backend yet.

## 2. Checkout — enroll

After `POST order /checkout` returns `fulfilled`, the frontend iterates
`session["pending_plans"]` and posts each one to:

```
POST pearcare-plan /enrollments
{ "user_id": ..., "plan_id": ..., "app_id": ..., "order_id": ... }
```

Failures are swallowed: a successful order should never be rolled back
because a warranty enrollment failed. *Known gap*: there is no retry
queue. A real implementation would persist the pending enrollments and
retry them out of band.

## 3. Account page — file a claim

The user picks an enrollment and types a free-text issue. The frontend
calls:

```
POST pearcare-claim /claims
{ "user_id": ..., "enrollment_id": ..., "issue": ... }
```

From there the claim is operator-triaged (or auto-triaged in this
demo). Replacement claims re-enter the storefront's fulfillment service
and the user's library updates as if they had bought the app a second
time.

## What the storefront does not know about PearCare

* Pricing math beyond the displayed `price_cents`. PearCare owns its
  own SKU prices.
* Triage rules.
* Coverage windows. The frontend trusts whatever the plan service
  returns.
* Any state about denied or in-progress claims; only the latest list
  of enrollments is rendered on the account page.

This narrow integration surface is the point — PearCare can change its
internals freely as long as those three endpoint contracts hold.

## Open Questions

The unresolved issues or areas that require SME input for the PearCare integration with the storefront include:

* Clarifying the consistency model and specific data stored in the cart database [S1](../database/catalog-db.md)
* Understanding the data stored in the account database, including cart information and user-specific data [S7](../database/account-db.md), [S12](../database/review-db.md)
* Determining the exact duration of data retention for orders in the failed state [S6](../architecture/data-flow-purchase.md)
* Resolving the issue of automatic refunds not being handled upon fulfillment failure [S10](../architecture/overview.md)

Additionally, further detail is needed regarding the specific data stored in the account database for each user [S12](../database/review-db.md).
