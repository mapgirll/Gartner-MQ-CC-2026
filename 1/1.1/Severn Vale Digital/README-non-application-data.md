# Non-Application Data Scenarios for Gartner Demo (Section 1.1)

**Fictional company for this demo pack:** **Severn Vale Digital** (UK West Midlands–inspired). Use **`severn-vale-*`** index names for manual uploads; Fleet custom integration uses **`severn_vale_digital`**. See **`SEVERN-VALE-DIGITAL.md`** and **`CUSTOM-INTEGRATION-FORM.md`**.

**Sample data calendar:** Time-series files under `sample-data/` are generated for **all of February 2026** (`2026-02-01` … `2026-02-28`). The scripted incident (**trial signup drop + Account Creation P99 spike**) is on **2026-02-19**. Regenerate with **`python3 scripts/generate_sample_data_feb2026.py`**.

Scenarios for ingesting **cost, sustainability, and business metrics** as non-application data. Use CSVs, JSON, or NDJSON with your platform's file/API ingest (e.g. Elastic Agent, Logstash, direct bulk API, or upload).

---

## 1. Cloud cost & resource usage (Cost metrics)

**Story:** "We correlate cloud spend with workload and environment so FinOps can see cost per app, team, and environment."

| Source idea | Data type | Ingest format | Key fields |
|-------------|-----------|---------------|------------|
| AWS Cost Explorer / Cost and Usage Report | Aggregated daily cost by service, tag | CSV or JSON | `date`, `service`, `region`, `cost_usd`, `usage_type`, `environment`, `team`, `project` |
| Azure Cost Management export | Same idea | CSV/JSON | `date`, `resource_group`, `meter`, `cost`, `subscription`, `department` |
| GCP Billing export | Same idea | CSV/JSON | `date`, `sku`, `project_id`, `cost`, `labels` |

**Demo angle:** Ingest once (or on schedule), then join with application telemetry (e.g. by `service`/`project`) to show cost per transaction or per error spike.

---

## 2. Carbon / sustainability metrics

**Story:** "We track carbon and energy so we can report on sustainability and tie it to applications and cloud regions."

| Source idea | Data type | Ingest format | Key fields |
|-------------|-----------|---------------|------------|
| Cloud carbon footprint (e.g. internal tool or API) | Estimated CO2 by service/region | JSON/CSV | `date`, `region`, `service`, `co2_kg`, `energy_kwh`, `scope` |
| Datacenter or facility power | Meter readings | CSV | `timestamp`, `facility`, `power_kwh`, `pue` |
| Internal sustainability report | Monthly rollups | CSV/JSON | `month`, `business_unit`, `co2_kg`, `renewable_pct`, `target_vs_actual` |

**Demo angle:** Overlay CO2 or energy trends with app traffic or deployment events to show "carbon cost of a release" or "impact of moving workload to a greener region."

---

## 3. Business & revenue metrics

**Story:** "We bring business KPIs into the same place as technical signals so we can see how incidents affect revenue or conversions."

| Source idea | Data type | Ingest format | Key fields |
|-------------|-----------|---------------|------------|
| Finance/BI export | Daily revenue, orders, conversion | CSV/JSON | `date`, `region`, `product_line`, `revenue`, `orders`, `conversion_rate` |
| CRM (e.g. Salesforce) | Opportunities, pipeline, support tickets | JSON/CSV | `date`, `stage`, `amount`, `region`, `product`, `ticket_count`, `sla_breaches` |
| Internal dashboard export | MRR, churn, NPS | CSV | `month`, `mrr`, `churn_rate`, `nps`, `segment` |

**Demo angle:** Time-align with incidents or SLO breaches to show "revenue dip during outage" or "support ticket spike when error rate increased."

---

## 4. SaaS / CRM (e.g. Salesforce) – “Events from other tools”

**Story:** "We ingest events from Salesforce (and other SaaS) so we can correlate pipeline and support with our platform health."

| Source idea | Data type | Ingest format | Key fields |
|-------------|-----------|---------------|------------|
| Salesforce report export | Opportunities, leads, cases | CSV/JSON | `created_date`, `stage`, `amount`, `account_id`, `owner`, `product` |
| Salesforce Events (Change Data Capture / Platform Events) | Near real-time | JSON/NDJSON | `event_type`, `timestamp`, `object_type`, `record_id`, `changed_fields` |
| Support case summary | Daily/weekly | CSV | `date`, `product`, `severity`, `count`, `avg_resolution_hours` |

**Demo angle:** Show pipeline value or case volume in the same dashboards as app errors or latency; demonstrate "events from other monitoring/tools" (1.1d).

---

## 5. Network & CSP metadata (network observability / CSP)

**Story:** "We enrich our view with network and cloud metadata so we can explain latency or cost by region, AZ, or network path."

| Source idea | Data type | Ingest format | Key fields |
|-------------|-----------|---------------|------------|
| VPC flow log summary | Bytes/packets by subnet, region | CSV/JSON | `date`, `region`, `az`, `subnet`, `bytes_in`, `bytes_out`, `allowed_denied` |
| Cloud asset inventory | Resources by tag, type, region | JSON | `collected_at`, `resource_id`, `type`, `region`, `tags`, `cost_estimate` |
| CDN or edge logs (aggregated) | Traffic by region, status | NDJSON | `timestamp`, `region`, `status_code`, `bytes`, `cache_hit` |

**Demo angle:** Combines "network observability" and "Cloud Service Provider" data (1.1e) with cost and performance.

---

## 6. Recommended “quick win” combo for the demo

Pick **2–3** to keep the narrative clear:

1. **Cloud cost (CSV)** – e.g. daily cost by service/team. Easy to generate and ingest; strong cost story.
2. **Sustainability (JSON)** – e.g. CO2/energy by region/service. Differentiates and ties to cost/region.
3. **Business/Sales (CSV or JSON)** – e.g. daily revenue or Salesforce pipeline. Shows "business metrics" and "events from other tools."

Use **same time range** (e.g. last 30–90 days) and **consistent dimensions** (e.g. `date`, `region`, `service` or `team`) so you can:
- Build dashboards that show cost, carbon, and revenue side-by-side.
- Correlate with application data (e.g. by `service` or `environment`) if you add OTel or eBPF later.

---

## Ingest options (high level)

- **CSV:** File beat, Logstash CSV filter, or one-off upload → index.
- **JSON/NDJSON:** Same pipelines, or direct bulk API; NDJSON is ideal for large or streaming uploads.
- **Scheduled:** Recurring export from cloud/BI/Salesforce → same pipeline on a schedule to simulate "ongoing ingest from multiple sources."

---

## Elasticsearch Observability: manual upload

If you're using **Elasticsearch Observability** and the built-in option to **manually upload CSV and JSON**, use the following so the data maps and displays correctly.

### Where to upload

- **Kibana:** **Stack Management** → **Upload** (or **Data** → **Upload** depending on your version), or use **Discover** → **Import** / **Upload** when creating a data view.
- Alternatively: **Machine Learning** → **Data Visualizer** → **Import data** for CSV/JSON upload with automatic field detection.

### Sample files and suggested index names

| File | Format | Suggested index name | Time field for dashboards |
|------|--------|----------------------|---------------------------|
| `sample-data/cloud-cost-daily.csv` | CSV | `severn-vale-cost` or `observability-cost` | `date` |
| `sample-data/sustainability-co2-daily.json` | JSON (array) | `severn-vale-sustainability` | `date` |
| `sample-data/sustainability-co2-daily.ndjson` | NDJSON | `severn-vale-sustainability` | `date` (use if array JSON isn’t split) |
| `sample-data/business-metrics-daily.ndjson` | NDJSON | `severn-vale-business-metrics` | `date` |
| `sample-data/salesforce-pipeline-export.csv` | CSV | `severn-vale-salesforce` | `date` |
| `sample-data/carbon-intensity-by-region.csv` | CSV | `severn-vale-carbon-intensity` | `effective_date` (snapshot `2026-02-01`) |
| `sample-data/geographic-traffic-orders-daily.ndjson` | NDJSON | `severn-vale-geo-daily` | `date` — use **`latitude`** / **`longitude`** as **`double`**; Maps: separate lat/lon fields (`GEOGRAPHIC-DATA.md`) |

Use consistent index name patterns (e.g. `severn-vale-*`) so you can create **one data view** over all demo data, or separate data views per source for clearer demo flows.

### Mapping tips for manual upload

1. **Date fields**  
   Ensure `date` (and any `timestamp`) is mapped as **date** type so the global time picker and time-based dashboards work. The upload UI usually detects `yyyy-MM-dd`; if not, adjust in **Stack Management** → **Index Management** → index → **Edit mapping** after the first upload.

2. **Numeric fields**  
   - Cost, revenue, CO2, energy, counts: should be **float** or **long** so aggregations and visualizations (sum, avg, etc.) work.
   - If the upload wizard infers them as keyword, change to numeric in the mapping or re-upload and confirm types.

3. **JSON array file**  
   `sustainability-co2-daily.json` is a single JSON array. The upload UI may expect either:
   - **One document per array element** (preferred): use the option that “splits array into documents” or upload as NDJSON (see below), or  
   - **One document per file**: less ideal for dashboards. If that’s the only option, use the NDJSON version instead.

4. **NDJSON**  
   For `business-metrics-daily.ndjson`, upload as **JSON** in the upload UI. Select “Newline delimited” or “NDJSON” if the option exists so each line becomes one document. If the UI only accepts “JSON”, you can paste or upload the file; some UIs accept NDJSON as valid JSON (one object per line).

### After upload

- Create **data view(s)** (Management → **Data Views**) that include the new index(es) and set the **time field** to `date`.
- Use **Discover** or **Dashboard** to build visualizations; cost, CO2, revenue, and pipeline value will then work with the time picker and filters (e.g. by `service`, `region`, `stage`).
- In Observability, this non-application data will sit alongside your app/OTel/eBPF data; you can correlate by time and, if you use consistent dimensions (e.g. `service`, `region`), by those fields in the same or linked dashboards.

Sample files in this folder:

- `sample-data/cloud-cost-daily.csv`
- `sample-data/sustainability-co2-daily.json` (or `sustainability-co2-daily.ndjson` for upload)
- `sample-data/business-metrics-daily.ndjson`
- `sample-data/salesforce-pipeline-export.csv`

Use or edit these to match your cluster’s mapping and time range; the table above aligns them with Elastic manual upload.
