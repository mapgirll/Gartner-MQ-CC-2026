# Elastic Observability: Build steps for the scripted demo

**Fictional company:** **Severn Vale Digital** (West Midlands–inspired). Quick reference: **`SEVERN-VALE-DIGITAL.md`**. Manual-upload indices use the **`severn-vale-*`** pattern; Fleet custom integration uses package **`severn_vale_digital`** (see **`CUSTOM-INTEGRATION-FORM.md`**).

All sample files live in the **`sample-data/`** folder next to this document (same directory as `README.md`, `STORYLINE-scripted-demo.md`, etc.). If you open this repo in Cursor/VS Code, paths are: **`sample-data/<filename>`** from the **Section1** project root.

Use **manual upload** (Stack Management → Upload, or **Machine Learning** → **Data Visualizer** → **Import data**) unless you ingest via Fleet (**`CUSTOM-INTEGRATION-FORM.md`**).

---

## Step 0: Confirm files on disk

These are the **only** data files in `sample-data/` (ignore `sample-data/README.md`):

| Filename | Format |
|----------|--------|
| `salesforce-lead-to-opportunity-daily.csv` | CSV |
| `salesforce-pipeline-export.csv` | CSV |
| `cloud-cost-daily.csv` | CSV |
| `carbon-intensity-by-region.csv` | CSV |
| `otel-account-creation-latency-daily.ndjson` | NDJSON (one JSON object per line) |
| `business-metrics-daily.ndjson` | NDJSON |
| `service-usage-by-region-daily.ndjson` | NDJSON |
| `sustainability-co2-daily.ndjson` | NDJSON |
| `sustainability-co2-daily.json` | JSON array of objects |
| `geographic-traffic-orders-daily.ndjson` | NDJSON (geo + sessions / leads / orders) |
| `geographic-traffic-orders-daily.csv` | CSV (same data; `latitude` / `longitude`) |

**Regenerate:** `python3 scripts/generate_sample_data_feb2026.py` (from the Section1 folder). Data range: **February 2026**; incident **19 Feb 2026**.

---

## Step 1: Upload files and index names

Upload each file you need. When the wizard asks for an index name, use the value below (or your own pattern—stay consistent).

### Minimum for the two dashboards in this guide

| # | File to upload (under `sample-data/`) | Suggested index name | Time field to set in data view |
|---|----------------------------------------|----------------------|--------------------------------|
| 1 | `salesforce-lead-to-opportunity-daily.csv` | `severn-vale-sf-funnel` | `date` |
| 2 | `otel-account-creation-latency-daily.ndjson` | `severn-vale-otel-summary` | `date` |
| 3 | `carbon-intensity-by-region.csv` | `severn-vale-carbon-intensity` | `effective_date` |
| 4 | `service-usage-by-region-daily.ndjson` | `severn-vale-usage-by-region` | `date` |

### Strongly recommended for Dashboard 1 (rich Funnel — not just 2 charts)

Upload these **in addition** to the funnel + OTel files so the SaaS-to-App dashboard can show pipeline + revenue context (same **`sample-data/`** paths):

| # | File | Suggested index name | Time field |
|---|------|----------------------|------------|
| 5 | `business-metrics-daily.ndjson` | `severn-vale-business-metrics` | `date` |
| 6 | `salesforce-pipeline-export.csv` | `severn-vale-salesforce-pipeline` | `date` |

### Optional (rest of demo / other dashboards)

| File under `sample-data/` | Suggested index name | Time field |
|---------------------------|----------------------|------------|
| `cloud-cost-daily.csv` | `severn-vale-cost` | `date` |
| `sustainability-co2-daily.ndjson` | `severn-vale-sustainability` | `date` |
| `sustainability-co2-daily.json` | `severn-vale-sustainability` (or a second index) | `date` — prefer **NDJSON** if the UI splits lines better than one array |
| `geographic-traffic-orders-daily.ndjson` | `severn-vale-geo-daily` | `date` |
| `geographic-traffic-orders-daily.csv` | `severn-vale-geo-daily` (same index or separate; see below) | `date` |

**Upload tips**

- **NDJSON:** In the import UI, choose newline-delimited / NDJSON if available so **each line** becomes one document.
- **carbon-intensity-by-region.csv:** Small reference table; **`effective_date`** is `2026-02-01` on every row.
- **Geographic data:** Use top-level **`latitude`** and **`longitude`** (see NDJSON). Map both as **`double`**. In **Elastic Maps**, choose **latitude + longitude as separate fields** — do not use a nested `location` object. Details: **`GEOGRAPHIC-DATA.md`**.
- Map **`date`** (and **`effective_date`**) as **date** where possible; map costs, latencies, **emissions_mt_co2e**, **request_volume** as **numeric** (float/long). Fix in **Index Management** if the wizard infers **keyword**.

---

## Step 2: Create data views

**Stack Management** → **Data Views** → **Create data view**.

### Required for Dashboards 1 & 2 (this guide)

| Data view name | Index pattern | Time field |
|----------------|---------------|------------|
| Severn Vale – Salesforce Funnel | `severn-vale-sf-funnel` | `date` |
| Severn Vale – OTel Summary | `severn-vale-otel-summary` | `date` |
| Severn Vale – Carbon Intensity | `severn-vale-carbon-intensity` | `effective_date` |
| Severn Vale – Usage by Region | `severn-vale-usage-by-region` | `date` |

### Recommended for Dashboard 1 (upload files 5–6 above)

| Data view name | Index pattern | Time field |
|----------------|---------------|------------|
| Severn Vale – Business metrics | `severn-vale-business-metrics` | `date` |
| Severn Vale – Salesforce pipeline | `severn-vale-salesforce-pipeline` | `date` |

### Optional (other dashboards)

| Data view name | Index pattern | Time field |
|----------------|---------------|------------|
| Severn Vale – Cloud cost | `severn-vale-cost` | `date` |
| Severn Vale – Sustainability CO2 | `severn-vale-sustainability` | `date` |
| Severn Vale – Geo traffic & orders | `severn-vale-geo-daily` | `date` |

Save each data view.

---

## Step 3: Dashboard 1 – SaaS-to-App Conversion Funnel

**Dashboard** → **Create dashboard** → name it **SaaS-to-App Conversion Funnel** (or **Severn Vale · Funnel & revenue**).

**Goal:** Look like an exec-ready “command center”: **CRM funnel**, **revenue/order KPIs**, **pipeline depth**, and **app performance** on one screen—not two lonely lines.

**Data views used:** **Severn Vale – Salesforce Funnel**, **Severn Vale – OTel Summary**, plus **Severn Vale – Business metrics** and **Severn Vale – Salesforce pipeline** if you followed the **strongly recommended** uploads in Step 1.

Set the dashboard **time range** to **February 2026** (or **1–28 Feb 2026**) once; every panel inherits it. Resize panels so the top has a title row, then **2–3 rows** of charts (use **Edit** → drag panel corners; typical layout is **48-column** width with **half-width** or **quarter-width** tiles).

---

### A. Title strip (optional but polished)

1. **Markdown** panel (if your version supports it on dashboards): short text, e.g.  
   **Severn Vale Digital** · *SaaS funnel + application performance* · sample data **Feb 2026** · incident **19 Feb**.

---

### B. CRM funnel row (same data view: Salesforce Funnel)

Use **Lens** (or **Aggregation-based**): data view **Severn Vale – Salesforce Funnel**, horizontal axis **date** (daily interval).

| # | Visualization | Vertical axis / metric | Title idea |
|---|----------------|-------------------------|------------|
| 2 | **Line** or **Area** | **trial_signups** (max or sum per day) | Trial signups (daily) |
| 3 | **Line** | **conversion_rate_pct** (max per day) | Conversion rate % (Funnel) |
| 4 | **Line** | **leads** (max per day) | Leads (volume) |
| 5 | **Area** or **Bar** | **mql_count** (max per day) | MQL count |

**Tip:** Put **trial_signups** and **conversion_rate_pct** **side-by-side** (same height)—the **19 Feb** dip shows up in both and reads as one story.

---

### C. Application performance row (OTel summary)

Data view: **Severn Vale – OTel Summary**. Filter: **`service_name` = `account_creation`** (if you have other services later).

| # | Visualization | Metric | Title idea |
|---|----------------|--------|------------|
| 6 | **Line** | **p99_latency_ms** (max per day) | Account Creation · P99 latency (ms) |
| 7 | **Line** | **p50_latency_ms** (max per day) | Account Creation · P50 latency (ms) |
| 8 | **Line** | **request_count** (max or sum per day) | Account Creation · Request volume |

**Tip:** Place **P99** wide (hero chart); **P50** and **request_count** can be **half-width** underneath or beside it. On **19 Feb**, P99 spikes and **request_count** drops—call that out live.

---

### D. Business KPIs row (needs **business-metrics** index)

Data view: **Severn Vale – Business metrics**.

9. **Multi-series line (Lens):** Horizontal axis **date** (daily). Vertical axis **value** (average—or max—per day). **Break down by** **`metric`** (you should see `revenue`, `orders`, `conversion_rate`, `mrr`).  
   - Title: **Daily business metrics (finance / product)**  
   - *Note:* `revenue`/`mrr` are large vs `orders`/`conversion_rate`; if the chart looks flat for small series, either **normalize** with two panels (one for **usd** metrics, one for **count/pct**) or use **two Lens charts** filtered by `metric` groups.

**Split option (often clearer on a big screen):**  
   - Panel 9a: filter `metric` is `revenue` **or** `mrr` → line **value** by **date**.  
   - Panel 9b: filter `metric` is `orders` **or** `conversion_rate` → line **value** by **date**.

---

### E. Pipeline depth (needs **salesforce-pipeline** index)

Data view: **Severn Vale – Salesforce pipeline**.

10. **Stacked area** or **Stacked bar** (Lens): Horizontal axis **date** (daily). Vertical axis **Sum of pipeline_value_usd**. **Break down by** **`stage`** (Discovery → Closed Won).  
    - Title: **Pipeline value by stage (USD)**  
    - Reads as “shape of the funnel in dollars” and fills the bottom of the dashboard nicely.

---

### F. Map — website traffic + orders by billing city (optional)

Requires **`geographic-traffic-orders-daily.ndjson`** (or CSV) → index **`severn-vale-geo-daily`** and data view **Severn Vale – Geo traffic & orders**. Details and narration: **`GEOGRAPHIC-DATA.md`**.

11. **Elastic Maps** panel on the same dashboard:  
    - **Layer type:** Documents (or clusters).  
    - **Index / data view:** **Severn Vale – Geo traffic & orders**.  
    - **Position:** **Latitude** = **`latitude`**, **Longitude** = **`longitude`** (two separate fields — not a combined geo field).  
    - **Metric:** e.g. **Sum** of **`website_sessions`**, **`orders_placed`**, or **`revenue_usd`**.  
    - **Tooltip:** **`city`**, **`country`**, **`market`**.  
    - **Time:** February 2026 — compare **18 Feb** vs **19 Feb** for the UK home-market story.

**Layout tip:** Full-width **Maps** row under the pipeline chart reads well as a “where the business felt it” closer.

---

### Minimum viable version

If you only uploaded the **four minimum** indices, build **panels 2–3** (trial signups + conversion %) and **panels 6–8** (P99 / P50 / request volume). The dashboard will still look **fuller than two panels**; add **9–10** when business + pipeline data is in.

**Demo beat:** On **19 Feb 2026**, align **conversion_rate_pct** + **trial_signups** with **P99** spike and lower **request_count**; optionally point at **revenue** / **orders** in business metrics the same day; on the **map**, show **UK home-market** cities cooling more than **NA/APAC** (see **`GEOGRAPHIC-DATA.md`**).

---

## Step 4: Dashboard 2 – Green Ops Sustainability

**Create dashboard** → name it **Green Ops Sustainability**.

Uses **`service-usage-by-region-daily.ndjson`** → `severn-vale-usage-by-region` and **`carbon-intensity-by-region.csv`** → `severn-vale-carbon-intensity`.

1. **Add panel – Emissions by service and region**  
   - **Vertical bar** or **Stacked bar**.  
   - Data view: **Severn Vale – Usage by Region**.  
   - X-axis: **service** (terms), split series by **region** (terms).  
   - Y-axis: **emissions_mt_co2e** (Sum).  
   - Title: `Emissions (mt CO2e) by service and region`.

2. **Add panel – Request volume by service and region**  
   - **Bar** or **Data table**.  
   - Data view: **Severn Vale – Usage by Region**.  
   - Rows / breakdown: **service**, **region**; metric: **request_volume** (Sum).  
   - Title: `Request volume by service and region`.

3. **Add panel (optional) – Carbon intensity by region**  
   - **Table**.  
   - Data view: **Severn Vale – Carbon Intensity**.  
   - Columns: **region** (or **region_label**), **g_co2_per_kwh**.  
   - Title: `Carbon intensity (g CO2/kWh) by region`.

4. Set **time range** to **February 2026** (usage data is daily **1–28 Feb 2026**). Save dashboard.

**Demo beat:** Compare **us-east-1** vs **eu-west-1** using **g_co2_per_kwh** (reference index) and **emissions_mt_co2e** / **request_volume** (usage index).

**Optional:** If you ingested **`sustainability-co2-daily.ndjson`**, add a panel from **Severn Vale – Sustainability CO2** (e.g. **co2_kg** by **date** and **region**). If you ingested **`cloud-cost-daily.csv`**, add cost from **Severn Vale – Cloud cost**.

---

## Step 5: Optional enrichment

- **Runtime field (Usage by Region):** Optional calculated **emissions** from **kwh** × **g_co2_per_kwh**; sample NDJSON already includes **emissions_mt_co2e**.
- **Unified dashboard:** Link Funnel + Green Ops and, if uploaded, **severn-vale-cost** / **severn-vale-sustainability**.
- **SLO:** If you have real OTel traces in Observability, add an SLO on Account Creation and pin it to the Funnel dashboard.

---

## Checklist before the demo

- [ ] **Step 0:** All expected files are present under **`sample-data/`** (see table above).  
- [ ] **Step 1:** Minimum four uploads for both dashboards; **plus** **`business-metrics-daily.ndjson`** + **`salesforce-pipeline-export.csv`** if you want the **full** Funnel layout (Step 3).  
- [ ] **Step 2:** Matching data views created; time fields **`date`** vs **`effective_date`** set correctly.  
- [ ] **Dashboard 1** built with **several panels** (funnel + OTel; add business + pipeline when uploaded); time range **Feb 2026**; incident **19 Feb 2026** visible.  
- [ ] **Dashboard 2** built; time range **Feb 2026**.  
- [ ] Narrative walkthrough: **`STORYLINE-scripted-demo.md`**.
