# payment

Charges money. Three providers. POST /authorize, POST /refund.

There are some hooks under hooks/. Stripe is the default. PayPal and Pearcard
also exist.

If the card is declined you get a 402 back. The payment id format is `pay_xxx`.

## Endpoints

<!-- SME-PLACEHOLDER:q-sd-adb396b490 START -->
> ⏳ **Waiting for SME** — *Topic:* Endpoints
>
> *Question:* What are the REST/RPC methods, paths, and handlers for the payment service?
> *Best guess (low-confidence):* The payment processing system's HTTP/RESTful API endpoints, methods, paths, and corresponding handler functions or controllers are not explicitly described in the provided sources. However, based on the interactions between services mentioned in [S1] and [S2], we can infer some information about the payment processing system.

The Order service interacts with the Payment service to orchestrate payments [S1]. This implies that there is a payment-related endpoint or method within the Order service's API. The path for this endpoint is not specified, but it likely involves a POST request to initiate a payment transaction.

Similarly, the Fulfillment Service (FUL) processes replacement orders and sends entitlements in response to POST requests from the CL service [S2]. This suggests that there may be an endpoint or method within the Order service's API for handling replacement orders and entitlements.

Further detail regarding the specific endpoints, methods, paths, and corresponding handler functions or controllers is TBD due to the lack of explicit information in the provided sources.
> *Asked:* on 2026-06-27 · *Status:* pending · *Question ID:* `q-sd-adb396b490`
<!-- SME-PLACEHOLDER:q-sd-adb396b490 END -->

## Data model

The key entities involved in the payment process are:

* **Payment**: owned by the payment service, which wraps three provider hooks (`stripe`, `paypal`, `pearcard`) [S1](../database/payment-db.md).
* **Order**: handled by the order service, which orchestrates interactions with the payment and fulfillment services [S4](order.md).
* **Cart**: managed by the cart service, which owns the cart database and is responsible for managing user carts, which are per-user, ephemeral, and idempotent [S3](cart.md).

The primary attributes of these entities are:

* **Payment**:
	+ Not explicitly mentioned in the provided sources (further detail TBD) [S6](../database/payment-db.md).
* **Order**:
	+ `id`: The unique ID of the order (integer) [S5](order.md).
	+ `user_id`: The ID of the user who placed the order (integer) [S5](order.md).
	+ `total_cents`: The total cost of the order (integer) [S5](order.md).
	+ `payment_id`: The payment ID associated with the order (string) [S5](order.md).
* **Cart**:
	+ `user_id`: The ID of the user who owns the cart (integer) [S3](cart.md).
	+ `items`: A list of items in the cart, each containing attributes like `app_id`, `quantity`, and `price_cents` (array of objects) [S3](cart.md).

The payment process involves interactions between the order service, payment service, and fulfillment service. The order service retrieves the cart contents from the cart service and uses this information to interact with the payment service to retrieve relevant data and complete the payment process [S4](order.md).
