# Sample data (Severn Vale Digital demo)

- **Range:** **2026-02-01** through **2026-02-28** (all of February 2026; 28 days).
- **Incident day:** **2026-02-19** — lower trial signups / conversion in Salesforce-style files, high **Account Creation** P99 in OTel summary, lower **account_creation** traffic in usage data.
- **Regenerate:** from repo root `Section1/`, run:

  ```bash
  python3 scripts/generate_sample_data_feb2026.py
  ```

- **Static reference:** `carbon-intensity-by-region.csv` — regional **g CO₂/kWh** with **`effective_date`** = `2026-02-01` (not a daily series).

| File | Rows / scale |
|------|----------------|
| `cloud-cost-daily.csv` | 28 days × 5 services |
| `salesforce-lead-to-opportunity-daily.csv` | 28 daily rows |
| `salesforce-pipeline-export.csv` | 28 days × 5 stages |
| `business-metrics-daily.ndjson` | 28 days × 4 metrics |
| `otel-account-creation-latency-daily.ndjson` | 28 daily rows |
| `sustainability-co2-daily.ndjson` / `.json` | 28 days × 4 region/service rows |
| `service-usage-by-region-daily.ndjson` | 28 days × 4 (2 services × 2 regions) |
| `geographic-traffic-orders-daily.ndjson` | 28 days × 13 cities; **`latitude`** / **`longitude`** (no nested `location`) |
| `geographic-traffic-orders-daily.csv` | Same as NDJSON; **`latitude`** / **`longitude`** columns |

See **`GEOGRAPHIC-DATA.md`** for map ingest and **19 Feb** behaviour.
