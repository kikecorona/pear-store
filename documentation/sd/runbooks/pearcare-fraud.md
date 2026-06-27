# pearcare fraud

TODO

## Overview

**Primary Characteristics of PearCare Scams**

PearCare scams exploit vulnerabilities in the integration between the storefront and PearCare services. The primary characteristics of these scams include:

* **Lack of transparency**: The storefront does not have access to pricing math, triage rules, or coverage windows, which are owned by PearCare [S1](../pearcare/integration.md).
* **Data inconsistencies**: The frontend trusts data returned from the plan service, but the consistency model and specific data stored in the cart database are unclear [S1](../pearcare/integration.md) (further detail TBD).
* **Claim lifecycle ownership**: The claim lifecycle (`filed → triaged → approved | denied`) is owned by the PearCare Claim service, which determines the resolution type and triggers corresponding hooks [S2](../architecture/overview.md), [S3](../pearcare/overview.md).
* **Triage rules simplicity**: Triage rules are intentionally simple and can be found in the `claim/app.py` file [S3](../pearcare/overview.md).

**Vulnerabilities Associated with PearCare Scams**

The vulnerabilities associated with PearCare scams include:

* **Lack of data retention**: The exact duration of data retention for orders in the `failed` state is not explicitly stated [S6](../pearcare/claim-service.md).
* **Dependence on Fulfillment Service**: The Fulfillment Service (FUL) reuses its logic from PearCare's claim service when a claim resolves to a replacement, making it a critical component in managing entitlements and replacements [S7](../pearcare/overview.md).

**Mitigation Strategies**

To mitigate these vulnerabilities, the storefront should:

* **Implement data validation**: Validate data returned from the plan service to ensure consistency with the cart database.
* **Develop clear triage rules**: Develop more complex triage rules that accurately determine resolution types.
* **Monitor data retention**: Establish a clear policy for data retention and implement measures to ensure compliance.

Note: Further detail regarding the specific data stored in the account database for each user is TBD [further detail TBD].

## Details

To address pearcare fraud, the following details should be included in this runbook:

* The claim lifecycle (`filed → triaged → approved | denied`) is owned by the **PearCare Claim** service, which determines the resolution type and triggers the corresponding hook [S1](../architecture/overview.md).
* For repair resolutions, the **PearCare Claim** service dispatches a request to the Repair Vendor Hook (RV) [S3](../architecture/data-flow-purchase.md), while for replacement resolutions, it sends a POST request to the Fulfillment Service (FUL) with the order ID and claim ID [S4](../database/pearcare-claim-db.md).
* The triage rules are intentionally simple, as specified in the `claim/app.py` file [S3](../architecture/data-flow-purchase.md).
* Data retention for orders in the `failed` state is handled by persisting orders in the `failed` state for audit purposes, although the exact duration of data storage is not explicitly stated [S6](integration.md).

The Fulfillment Service (FUL) reuses its logic from PearCare's claim service when a claim resolves to a replacement, suggesting that the fulfillment service plays a key role in managing entitlements and replacements [S7](../database/cart-db.md).

## Open Questions

Open questions or areas of uncertainty regarding pearcare fraud that require SME input include:

* Clarifying the consistency model and specific data stored in the cart database [S1](../pearcare/integration.md)
* Understanding the data stored in the account database for each user, particularly with regards to denied or in-progress claims [S2](../pearcare/overview.md), [S5](../architecture/data-flow-purchase.md)
* Determining the exact duration of data retention for orders in the `failed` state [S6](../pearcare/overview.md)

Additionally, there are areas where further detail is TBD:

* The specific data stored in the account database for each user [further detail TBD]
* The triage rules and how they interact with the claim lifecycle [S3](../database/pearcare-claim-db.md), [S4](../pearcare/overview.md)
