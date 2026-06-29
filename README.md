## Implementation & documentation gap (synthetic data)

This project contains a small Pear Store storefront — eight
backend microservices plus a sibling PearCare warranty system — used
as input for the Research Agent's gap-detection workflows. Both the
**implementation** and the **documentation** are intentionally uneven
so the agent has gaps to find:

| Surface       | Where                                       | Uneven dimensions                                                                  |
| ------------- | ------------------------------------------- | ---------------------------------------------------------------------------------- |
| Service code  | `implementation/services/`   | docstrings range from thorough to one-liner-with-typos; some hooks are stubs        |
| Service docs  | `documentation/sd/services/` | HIGH / MEDIUM / LOW / VERY LOW — `documentation/README.md` is the gap map           |
| DB docs       | `documentation/sd/database/` | three docs MISSING outright (`order`, `fulfillment`, `pearcare-plan`)               |
| Telemetry     | `telemetry/` + `documentation/sd/telemetry/` | five services instrumented at different tiers (HIGH/MEDIUM/LOW), five emit nothing  |
| Runbooks      | `documentation/sd/runbooks/` | `pearcare-fraud.md` is a placeholder; `incident-response.md` is one-liner thin      |
| Business docs | `documentation/bp/business-cases/` | refunds / piracy / cancellations are deliberately under-developed                 |

## Telemetry coverage tiers

Five backend services are wired to a small dependency-free OpenTelemetry
shim (`implementation/shared/otel.py`) that emits
OTLP-shaped JSONL into `telemetry/<service>/` while
the services run:

- **HIGH** — `order`, `payment`: root spans, child spans for every
  downstream call, success/failure counters, latency histograms, and
  business metrics.
- **MEDIUM** — `fulfillment`, `pearcare-claim`: root spans + standard
  metrics + one business counter, but the inner hook calls aren't
  spanned (deliberate gap).
- **LOW** — `catalog`: counters and a latency histogram, **no traces**.
- **NONE** — `account`, `cart`, `review`, `search`, `pearcare-plan`:
  uninstrumented (deliberate gap; the doc gap agent should flag the
  five missing telemetry docs).

The doc tree mirrors the same shape: `documentation/sd/telemetry/order.md`
is the HIGH-quality template, and `documentation/sd/telemetry/search.md`
is intentionally wrong (claims metrics that don't exist) so the
verifier has something concrete to reconcile with code.

To produce telemetry, boot the stack with
`implementation/start_all.sh`, drive some traffic, and
read the JSONL files at `telemetry/<service>/`. Stop
the stack with `implementation/stop_all.sh`.
