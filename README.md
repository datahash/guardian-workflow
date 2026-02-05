Integrate **Hedera Guardian GHGP Corporate Standard V2 policy** into the workflow so you can push event-ledger data into the policy, compute emissions *in compliance with recognized standards*, and then issue **Carbon Emission Tokens (CETs)**. The policy follows the **Greenhouse Gas Protocol Corporate Standard** (including Scope 2 Guidance) and defines how MRV data should be structured, validated, and tokenized.

---

## 🌍 End-to-End Workflow Including GHGP Corporate Standard V2

### 1) Event Ledger → Data Extraction

Extract daily resource usage from your **event ledger**:

| Metric                    | Unit | Notes                |
| ------------------------- | ---- | -------------------- |
| Essential electricity     | kWh  | e.g., plant power    |
| Non-essential electricity | kWh  | e.g., refrigeration  |
| Total water               | m³   | water consumption    |
| Hot water                 | m³   | sub-portion of total |
| Gas (low pressure)        | m³   | fuel source          |

Each record should be

* timestamped
* uniquely hashed
* linked to a facility or asset in your ledger

---

### 2) Normalisation & Emissions Calculation

Before feeding into Guardian, normalize metrics and compute carbon emissions using GHGP-aligned factors. The **GHGP Corporate Standard V2 policy** expects:

✔ Scope 1: direct combustion sources (e.g., gas)
✔ Scope 2: purchased electricity (location- or market-based)
✔ Optionally Scope 3 if needed later

‼ Important
GHGP Scope 2 Guidance requires that *electricity activity data (kWh)* be multiplied by appropriate emission factors, and both *location-based* and *market-based* accounting are supported when available.

**Example internal transformation**

```json
{
  "facility_id": "F-001",
  "date": "2026-02-05",
  "electricity_kwh": 600,
  "water_m3": 12.5,
  "hot_water_m3": 4.2,
  "gas_m3": 38,
  "scope1_co2e_t": ...,
  "scope2_co2e_t": ...
}
```

---

### 3) Prepare GHGP-Aligned MRV Payload

Guardian’s GHGP v2 policy uses a **hierarchical entity → asset → source → raw data** model:

✔ **Organization Profile** – company metadata
✔ **Entity Schema** – business unit / subsidiary
✔ **Asset Schema** – facility or equipment
✔ **Source Schema** – e.g., electricity meter, gas meter
✔ **Raw Data Schema** – actual MRV values for metrics
✔ **Reporting Metrics** – computed emissions aligned with GHGP

Your normalized measurements become MRV records under the “Source” schema.

---

### 4) Guardian Policy Ingestion & Validation

**Process:**

1. **Authenticate** against Guardian using DID or API key
2. **Submit MRV data** as a policy input
3. **Policy executes auto-calculations:**

   * Applies GHGP emission factor tables
   * Maps raw activity to Scope 1/Scope 2 emissions
   * Checks completeness and structure
4. **Policy generates “Reporting Metrics”** — each linked to GHGP categories

**Optional:** A VVB (validation & verification body) can review the data before issuance, per GHGP workflow.

---

### 5) Token Issuance Logic

The **GHGP Corporate Standard V2 policy** defines a *Carbon Emission Token (CET)* equal to **1 metric tonne CO₂e**.

Issuance triggers when:

* normalized activity data is submitted and valid
* reporting metrics computed by the policy are verified (optional)
* the organization requests issuance for the reporting period

**Outcome:**
Guardian mints CET tokens based on total Scope 1 + 2 emissions computed under GHGP rules.

---

### 6) Transparent Audit Chain

Guardian preserves:

* Raw MRV inputs
* Policy execution logs
* Emission calculations
* Token minting transactions

This creates a **verifiable trust chain** compliant with reporting standards — ideal for ESG metrics, CSRD disclosures, or SBTi alignment.

---

## 🧠 Recommended Practices

✔ **Audit emission factors:** Pull up-to-date grid or supplier specs
✔ **Model scope boundaries carefully:** Define whether your meters map to organization asset boundaries authorized for GHGP reporting
✔ **Automate ingestion:** APIs or IoT devices to populate MRV data
✔ **Use hierarchical entity structure:** Align with GHGP corporate boundary rules
