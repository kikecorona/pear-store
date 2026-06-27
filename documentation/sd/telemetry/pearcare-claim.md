# pearcare-claim — telemetry (MEDIUM)

PearCare's claim service emits per-request signals plus a counter
that tracks triage decisions. The hook layer (`repair_vendor_hook`,
`replacement_hook`) is **not** wrapped in spans.

## Spans

Root spans only:

- `pearcare-claim POST /claims`
- `pearcare-claim POST /claims/<claim_id>/triage`
- `pearcare-claim POST /claims/<claim_id>/approve`
- `pearcare-claim POST /claims/<claim_id>/deny`
- `pearcare-claim GET /claims/<claim_id>`
- `pearcare-claim GET /users/<user_id>/claims`

## Metrics

| Name                                           | Kind      | Attributes                                            |
| ---------------------------------------------- | --------- | ----------------------------------------------------- |
| `pearcare-claim_requests_total`                | counter   | `http.method`, `http.route`, `http.status_code`, `status` |
| `pearcare-claim_request_duration_ms`           | histogram | same                                                  |
| `pearcare_claim_outcomes_total`                | counter   | `resolution` ∈ `{repair, replacement, support, deny}`, `tier` ∈ `{pearcare_basic, pearcare_plus, pearcare_loss}` |

`pearcare_claim_outcomes_total` increments on `/triage` after the
auto-triage decision is made. There is **not** a separate counter
on `/approve`, so the volume of approvals is implied from the
request counter.

## Known gaps

- The `repair_vendor_hook` and `replacement_hook` calls inside
  `/approve` are not spanned. A long approve span = a slow vendor
  call, but you cannot tell *which* vendor without going to the
  vendor system.
- `/approve` outcomes (vendor dispatched vs replacement issued)
  are not counted — they have to be inferred from the response body.
- `pearcare_claim_outcomes_total` uses an underscore-style metric
  name (`pearcare_claim_…`) while the request-level metrics use
  the hyphenated service name (`pearcare-claim_…`). This is a
  consistency wart; fix it together.

## Details

The per-request signals and counter that tracks triage decisions emitted by pearcare-claim involve several key endpoints and stages.

When a user initiates a claim, the frontend (`FE`) triggers a POST request to the **PearCare Claim** service (`CL`), which determines the resolution type (support, repair, replacement, or deny) based on the triage rules [S1](../architecture/data-flow-purchase.md). The triage rules are intentionally simple and can be found in the `claim/app.py` file [S3](../architecture/data-flow-pearcare-claim.md).

For repair resolutions, the `CL` service dispatches a request to the Repair Vendor Hook (`RV`) with vendor, ticket, and eta_days information [S1](data-flow-purchase.md). For replacement resolutions, the `CL` service sends a POST request to the Fulfillment Service (`FUL`) with the order ID and claim ID [S4](../architecture/data-flow-purchase.md).

The key metrics and their attributes are:

* **Resolution type**: The resolution type (support, repair, replacement, or deny) determined by the `CL` service based on the triage rules.
* **Triage rules**: The intentionally simple rules used to determine the resolution type, found in the `claim/app.py` file [S3](../architecture/data-flow-pearcare-claim.md).
* **Repair vendor hook**: The request sent to the Repair Vendor Hook (`RV`) for repair resolutions, which returns vendor, ticket, and eta_days information.
* **Fulfillment service**: The POST request sent to the Fulfillment Service (`FUL`) for replacement resolutions with the order ID and claim ID.

Note that further detail regarding the specific data stored in the account database for each user is TBD [further detail TBD].

## Open Questions

SMEs may have questions or concerns about the telemetry flow for pearcare-claim regarding:

* The exact duration of data retention for orders in the `failed` state [S6](../database/review-db.md).
* The specific data stored in the account database for each user, particularly with regards to cart information and order status [S5](../database/pearcare-claim-db.md), [S3](../database/pearcare-claim-db.md).
* The consistency model and specific data stored in the cart database, which may lead to data inconsistencies [S9](../runbooks/pearcare-fraud.md) (further detail TBD).
* The transparency of pricing math, triage rules, or coverage windows, which are owned by PearCare but not accessible to the storefront [S9](../runbooks/pearcare-fraud.md).
