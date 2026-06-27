# Incident response

If something is broken, page someone.

If checkout is broken, look at order-svc logs first.

If payments are failing, look at payment-svc.

Refunds are manual.

## Details

When an incident occurs, follow these steps:

1. Triage the issue using the triage rules to categorize and prioritize customer issues based on keywords [S1](data-flow-pearcare-claim.md).
2. If the issue requires a payment processing step, use the PAY service to process payments, which returns a payment ID and charge ID upon successful payment [S2](data-flow-purchase.md).
3. For order fulfillment, create entitlements for licenses and downloads using the Fulfillment Service (FUL) [S2](data-flow-purchase.md).
4. If the issue involves cart management, clear the customer's cart and create an order in the failed state if payment is declined or fulfillment fails [S2](data-flow-purchase.md).

For checkout and payment services:

1. The Order Service (ORD) initiates a POST request to the Payment Service (PAY) with the order ID and amount cents for authorization [S6](data-flow-purchase.md).
2. The PAY service returns a 200 response with the payment ID and charge ID upon successful payment [S6](data-flow-purchase.md).
3. The ORD service then initiates a POST request to the Fulfillment Service (FUL) with the order ID, user ID, and app IDs for fulfillment [S6](data-flow-purchase.md).
4. The FUL service processes replacement orders and sends entitlements in response [S3](claim-service.md), [S4](data-flow-purchase.md).

Note that automatic refunds are not handled upon fulfillment failure, requiring human intervention to run a POST request to the Order Service (ORD) with the order ID for refund [S10](architecture/overview.md).

## Open Questions

There are open issues and areas where SME input is required to improve the incident response process. One known gap is that automatic refunds are not handled upon fulfillment failure, requiring human intervention [S2](../architecture/data-flow-purchase.md). Additionally, there is a pending question regarding what non-goals or out-of-scope work are explicitly excluded from this purchase data flow [S1](../database/cart-db.md).

The system's architecture and design decisions have been driven by the need for modularity and decoupling of services, allowing for independent development and maintenance [S4](../pearcare/overview.md). However, the exact duration of data retention for orders in the `failed` state is not explicitly stated [S6](../pearcare/claim-service.md), which may be an area where SME input is required.

Furthermore, there are several areas where SME input could improve the incident response process:

* Refining the triage rules to better categorize and prioritize customer issues based on keywords [S3](../architecture/overview.md)
* Improving payment processing by handling declined payments more efficiently
* Enhancing order fulfillment by automating refunds upon fulfillment failure

Overall, while the system has made significant progress in modularity and decoupling of services, there are still areas where SME input is required to improve incident response processes.
