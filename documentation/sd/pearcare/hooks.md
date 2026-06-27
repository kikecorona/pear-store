# PearCare hooks

There are two hooks. They are how the claim service does anything in
the real world.

## `replacement_hook.replace(claim_id, user_id, app_id, original_order_id)`

Mints a fresh entitlement on the same `app_id` by calling the main
fulfillment service:

```
POST {fulfillment}/fulfill
{
  "order_id": "replacement-<claim_id>",
  "user_id":  <user_id>,
  "app_ids":  [<app_id>]
}
```

The synthetic order id is what later distinguishes a replacement from a
real purchase in the receipt store. The user's existing license stays
in place — having two licenses for the same app is fine in PearCare's
data model.

## `repair_vendor_hook.dispatch(...)`

It picks a vendor and gives back a ticket. We don't really call any vendor.
TODO: write this up.

## Details

The key differences between the replacement_hook and repair_vendor_hook are their functions and outputs.

replacement_hook is responsible for replacing a claim, which involves sending a POST request to the Fulfillment Service (FUL) with an order ID for replacement [S2](../architecture/data-flow-purchase.md). The FUL service then responds with entitlements. This hook calls fulfillment-svc to mint a fresh license + downloads, resulting in a new entitlement appearing in the user's library [S1](claim-service.md).

repair_vendor_hook, on the other hand, dispatches repair requests to vendors and returns {vendor, ticket, eta_days} which is attached to the claim [S1](claim-service.md).

The use cases for these hooks are as follows:

* replacement_hook: used when the resolution of a claim is "replacement", allowing the user to receive a new entitlement.
* repair_vendor_hook: used when the resolution of a claim is "repair", dispatching the repair request to a vendor and providing eta_days information.

Note that the Fulfillment Service (FUL) is responsible for administering the pearcare claim database [S4](../database/pearcare-claim-db.md), but it also processes replacement orders as part of its fulfillment service [S2](../architecture/data-flow-purchase.md).

## Open Questions

To integrate the repair vendor hook, the subsequent actions are as follows:

1. The CL service dispatches a request to the Repair Vendor Hook (RV) with the claim ID, app ID, and issue [S4](../architecture/data-flow-pearcare-claim.md).
2. The RV returns vendor, ticket, and eta_days information [S4](../architecture/data-flow-pearcare-claim.md).

Suppliers must provide the following data or documentation:

* Vendor information
* Ticket details
* Estimated time of arrival (eta_days)

Note that the repair resolution type is determined by the CL service based on the triage rules, which are intentionally simple and should be replaced with real eligibility logic [S4](../architecture/data-flow-pearcare-claim.md).
