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

---

## Stage 0 — Foundations & Decisions (do this first)

### 0.1 Define reporting boundaries

**Tasks**

* Define **organizational boundary** (legal entity vs operational control)
* Define **facility boundary** (what meters belong to which facility)
* Decide reporting cadence (daily ingestion → monthly issuance is common)

**Outputs**

* Boundary definition document
* Facility & meter registry (IDs, location, scope mapping)

---

### 0.2 Decide Scope handling (per GHGP V2)

**Tasks**

* Scope 1 → Gas (low pressure)
* Scope 2 → Electricity (essential + non-essential)
* Decide:

  * Location-based only **or**
  * Location + market-based (if supplier data exists)

**Outputs**

* Scope mapping table
* Emission factor source list (grid, gas, water)

---

## Stage 1 — Event Ledger Extension (you own this)

You **do not** change the ledger’s core role. You extend it.

### 1.1 Add emissions-ready data model

**Tasks**

* Ensure ledger events include:

  * Facility ID
  * Meter/source ID
  * Timestamp (ISO-8601, UTC)
  * Raw units (kWh, m³)
* Ensure **immutability + hash generation**

**Outputs**

* Versioned event schema
* Ledger event hash per record

---

### 1.2 Embed Normalisation & Calculation Component

You said this can live *inside* the ledger app — perfect.

**Tasks**

* Unit normalization:

  * Electricity → kWh
  * Water → m³
  * Gas → m³
* Derived calculations:

  * Hot water energy (ΔT × volume)
* Apply **GHGP-aligned emission factors**
* Separate:

  * Raw activity data
  * Calculated emissions

**Outputs**

* Normalised activity dataset
* Calculated emissions dataset
* Deterministic calculation logic (versioned)

---

### 1.3 Evidence packaging

**Tasks**

* Generate an **evidence bundle** per reporting period:

  * Ledger event hashes
  * Calculation version
  * Emission factor version
* Store bundle hash on your side

**Outputs**

* Evidence hash (this is what Guardian anchors)

---

## Stage 2 — Guardian Environment Setup (Managed Service)

### 2.1 Guardian tenant configuration

**Tasks**

* Confirm Managed Guardian endpoints
* Create Guardian users & roles:

  * Organization Admin
  * Data Submitter
  * (Optional) Verifier/VVB

**Outputs**

* Guardian identities
* API credentials / DID mappings

---

### 2.2 Import GHGP Corporate Standard V2 policy

**Tasks**

* Load **GHGP Corporate Standard V2** policy
* Review policy blocks:

  * Organization
  * Facility (Asset)
  * Emission Sources
  * Raw Data
  * Reporting Metrics
* Freeze a **policy version** for production

**Outputs**

* Active GHGP V2 policy instance
* Policy version ID

---

## Stage 3 — Data Mapping Layer (critical glue)

This is where most projects succeed or fail.

### 3.1 Map ledger → Guardian schemas

**Tasks**

* Map:

  * Facility → Guardian Asset
  * Meter → Guardian Source
  * Daily usage → Raw Data schema
* Explicitly map:

  * Essential electricity → Scope 2
  * Non-essential electricity → Scope 2 (separate source)
  * Gas → Scope 1
  * Water → Optional reporting metric

**Outputs**

* Mapping specification document
* JSON payload templates per schema

---

### 3.2 Build Guardian ingestion client

**Tasks**

* Implement Guardian API client:

  * Authentication
  * Schema creation (once)
  * Raw data submission (recurring)
* Handle:

  * Idempotency
  * Retry logic
  * Submission status tracking

**Outputs**

* Guardian ingestion service/module
* Submission logs

---

## Stage 4 — Policy Execution & Validation Flow

### 4.1 Automated policy execution

**Tasks**

* Trigger policy after:

  * Reporting period close
  * All facilities submitted
* Monitor:

  * Validation failures
  * Missing sources
  * Unit mismatches

**Outputs**

* Policy execution records
* GHGP-compliant reporting metrics

---

### 4.2 Optional verification step (future-proof)

**Tasks**

* Enable VVB review workflow (optional now)
* Define approval thresholds (auto vs manual)

**Outputs**

* Verifier decision logs
* Audit-ready trail

---

## Stage 5 — Token Issuance Design

### 5.1 Token model (recommended)

**Carbon Emission Token (CET)**

* 1 token = 1 tCO₂e
* Fungible (HTS)
* Issued per org per reporting period

**Tasks**

* Confirm token metadata
* Define issuance trigger in policy

**Outputs**

* Token definition
* Issuance rules locked

---

### 5.2 Token lifecycle controls

**Tasks**

* Define:

  * Holding accounts
  * Retirement mechanism
  * (Optional) Internal transfers
* Prevent double issuance for same period

**Outputs**

* Token governance rules
* Retirement flow

---

## Stage 6 — Operations, Monitoring & Governance

### 6.1 Monitoring & alerts

**Tasks**

* Monitor:

  * Ledger → Guardian drift
  * Failed submissions
  * Policy changes
* Alert on:

  * Missing days
  * Anomalous consumption

**Outputs**

* Ops dashboards
* Alerting rules

---

### 6.2 Versioning & audit readiness

**Tasks**

* Version:

  * Emission factors
  * Calculation logic
  * Guardian policy
* Snapshot reports per period

**Outputs**

* Immutable audit snapshots
* Regulator-ready evidence pack

---

## End-State Architecture (What You’ll Have)

* **Ledger** → system of record
* **Normalization component** → deterministic MRV engine
* **Guardian** → trust, policy, and issuance layer
* **Tokens** → cryptographic proof of emissions
* **Full audit chain**:

  ```
  Raw event → hash → calculation → Guardian policy → token
  ```

---
