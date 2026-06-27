# payment — telemetry (HIGH)

Wired in `synthetic-data/implementation/services/payment/app.py`:

```python
TRACER, METER = instrument_flask(app, "payment")
auth_outcomes   = METER.counter("payment_authorize_outcomes_total")
refund_outcomes = METER.counter("payment_refund_outcomes_total")
amount_cents    = METER.histogram("payment_amount_cents")
```

## Spans

### Root spans

| `name`                                | When it fires                                |
| ------------------------------------- | -------------------------------------------- |
| `payment POST /authorize`             | `/authorize` — picks a provider, charges     |
| `payment POST /refund`                | `/refund` — looks up the payment, refunds    |
| `payment GET /payments/<pid>`         | read                                         |
| `payment GET /health`                 | health probe                                 |

`status=ERROR` when the provider declines (HTTP 402) or the payment
record is missing (HTTP 404).

### Child spans

One per provider call:

| `name`                                | When it fires                                | Attributes                                |
| ------------------------------------- | -------------------------------------------- | ----------------------------------------- |
| `provider.stripe.authorize`           | inside `/authorize` for a `tok_*` token      | `provider="stripe"`, `amount_cents`       |
| `provider.paypal.authorize`           | inside `/authorize` for a `pp_*` token       | `provider="paypal"`, `amount_cents`       |
| `provider.pearcard.authorize`         | inside `/authorize` for a `pc_*` token       | `provider="pearcard"`, `amount_cents`     |
| `provider.<name>.refund`              | inside `/refund` for the matching provider   | `provider`, `payment_id`                  |

If the provider declines the charge, the child span is `status=ERROR`
and carries `error="card_declined"` (or whichever string the hook
returned).

## Metrics

| Name                                       | Kind        | Attributes                                                       |
| ------------------------------------------ | ----------- | ---------------------------------------------------------------- |
| `payment_requests_total`                   | counter     | `http.method`, `http.route`, `http.status_code`, `status`        |
| `payment_request_duration_ms`              | histogram   | same                                                             |
| `payment_authorize_outcomes_total`         | counter     | `provider`, `outcome` ∈ `{authorized, declined}`                 |
| `payment_refund_outcomes_total`            | counter     | `provider`, `outcome` ∈ `{refunded}`                             |
| `payment_amount_cents`                     | histogram   | `provider`. **Recorded only on successful authorizations.**      |

## Notes on the provider attribute

`provider` is derived from the token prefix:

```
tok_*  → stripe
pp_*   → paypal
pc_*   → pearcard
(none) → random pick (synthetic only)
```

This means a request without an explicit `token` will round-robin
across providers, which is **not** a realistic production behaviour;
in the demo it is what gives the synthetic data its provider mix.

## Trace propagation

`payment` reads inbound `traceparent` headers and stitches into the
caller's trace, so a checkout from `order` produces a single
multi-service trace. Outbound calls — e.g. provider hooks — are
in-process and don't need propagation.

## Failure semantics

| Situation                  | Span `status` | `attributes.status` | HTTP code | Counter                                            |
| -------------------------- | ------------- | ------------------- | --------- | -------------------------------------------------- |
| Authorize OK               | OK            | success             | 200       | `payment_authorize_outcomes_total{outcome=authorized}` |
| Authorize declined         | ERROR         | failure             | 402       | `payment_authorize_outcomes_total{outcome=declined}`   |
| Authorize unknown token    | ERROR         | failure             | 402       | declined (provider rejects)                        |
| Refund OK                  | OK            | success             | 200       | `payment_refund_outcomes_total{outcome=refunded}`  |
| Refund of unknown payment  | ERROR         | failure             | 404       | (none)                                             |

## Known gaps

- A 4xx on `/refund` for an unknown payment id has no outcome
  counter — the gap agent should suggest one (e.g. an
  `outcome=not_found` value on the existing counter).
- Latency is not bucketed per provider. Stripe is consistently fast
  (~30 ms p50) and PearCard is consistently slow (~55 ms p50), but
  that is only visible if you slice the histogram by `provider`.

## Details

Here's a description of the flow of a payment request from the `/authorize` endpoint to the provider hooks:

When a user initiates a checkout, the Order Service retrieves the cart contents from the Cart Service [S3](cart.md). The Order Service then interacts with the Payment Service to authorize the payment for the order [S3](order.md) and retrieve relevant data [S5](../database/payment-db.md).

The Payment Service wraps three provider hooks (stripe, paypal, pearcard) [S1](../architecture/overview.md), and the Order Service initiates a POST request to the Payment Service with the order ID and amount cents for authorization [S6](data-flow-purchase.md). The PAY service returns a 200 response with the payment ID and charge ID upon successful payment [S6](data-flow-purchase.md).

The Payment Service then interacts with the provider hooks (stripe, paypal, pearcard) to complete the payment process. If the card is declined, a 402 error is returned [S3](payment.md). The payment id format is `pay_xxx` [S3](payment.md).

Relevant attributes and metrics include:

* Order ID (`id`) [S5](order.md)
* User ID (`user_id`) [S5](order.md)
* Total cost of the order (`total_cents`) [S5](order.md)
* Payment ID (`payment_id`) [S5](order.md)

Note that further detail regarding how the Search service uses this information is TBD.

## Open Questions

For a 4xx error on the `/refund` endpoint, the expected behavior is that the order remains in its current state, as the refund process is best-effort and does not guarantee success [S3](../services/order.md). The outcome counter for this scenario would be `payment_failed`, which collapses failures from different payment providers (e.g., Stripe, PayPal, PearCard) together [S1](order.md).

In terms of handling the error, since there is no business-level outcome counter for refunds [S1](order.md), a new counter should be proposed. Additionally, it's worth noting that on fulfillment failure, the order is currently left in `paid` state, and auto-rollback is a known gap [S4](../services/order.md).
