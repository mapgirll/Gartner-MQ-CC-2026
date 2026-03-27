# Scripted Storyline: SaaS Funnel + Green Ops (Elastic Observability)

Two scripted scenarios you can run back-to-back. Each has: **setup (ingest)**, **narrative beats**, **what to show**, and **dashboard/view build steps**.

**Fictional company:** **Severn Vale Digital** — use this name when you narrate business context (e.g. “Severn Vale’s trial signups dropped…”). Indices: **`severn-vale-*`**. Fleet custom integration package: **`severn_vale_digital`**. See **`SEVERN-VALE-DIGITAL.md`** and **`CUSTOM-INTEGRATION-FORM.md`**.

---

## Prerequisites

- Elasticsearch + Kibana (Observability).
- Data ingested via **manual upload** (CSV/JSON) as in `README-non-application-data.md`.
- Optional but recommended: at least one OTel or eBPF data source so you can reference “real” traces; the storyline works with **synthetic** trace-summary data if you don’t have live OTel yet.

---

# ACT 1: SaaS-to-App Conversion Funnel

**One-line pitch:** “We’re not just reporting an error—we’re reporting lost revenue.”

## Story in one sentence

Salesforce shows a drop in trial signups; we correlate it with OpenTelemetry showing the **Account Creation** service had high P99 latency that same day—so we connect **why** signups dropped (slow app) with **what** the business cares about (revenue).

## Data you need (ingest order)

| Order | File | Index name | Purpose |
|-------|------|------------|--------|
| 1 | `sample-data/salesforce-lead-to-opportunity-daily.csv` | `severn-vale-sf-funnel` | Trial signups, conversion rate, leads by day |
| 2 | `sample-data/otel-account-creation-latency-daily.ndjson` | `severn-vale-otel-summary` | Daily P99/P50 for “Account Creation” (synthetic OTel-style summary) |

**Elastic ingest:** Use **Stack Management → Upload** (or **Data Visualizer → Import**). Create a **data view** per index with **time field = `date`**.

## Scripted narrative (what to say)

1. **Setup**  
   *“We ingest data from multiple sources in one platform: Salesforce for the business funnel and OpenTelemetry for application performance.”*

2. **Show the business problem**  
   *“In Salesforce we see a clear drop in trial signups on 19 February 2026. Conversion rate dips. The business question is: why?”*

3. **Correlate with app data**  
   *“In the same platform we have OTel data. When we look at the Account Creation service, we see P99 latency spiked that same day. So we’re not just reporting an error—we’re reporting lost revenue: the slow signup flow directly lines up with the drop in trials.”*

4. **Value prop**  
   *“The platform understands the ‘why’ behind the ‘what’: one place where business metrics from Salesforce and application metrics from OpenTelemetry meet.”*

## What to show (click path)

1. Open **Discover** (or the **SaaS-to-App Funnel** dashboard).
2. Select data view that includes **Salesforce funnel** index. Filter or time-range to **February 2026** (or **1–28 Feb 2026**). Show **trial_signups** and **conversion_rate** over time; point out the **drop on 19 Feb 2026**.
3. Switch to (or add a panel for) **OTel summary** data view. Same time range. Show **P99 latency** for service **account_creation**; point out the **spike on 19 Feb 2026**.
4. Optional: use a **single dashboard** with two panels (Salesforce metrics + latency by day) and a shared time picker so the correlation is visible at a glance.

## Key metrics for the storyline

- **salesforce_conversion_rate** (or **trial_signups**) from Salesforce CSV.
- **app_checkout_latency_ms** or **account_creation_p99_latency_ms** from OTel/summary data.

Use the same **date** axis so the drop and the spike align on **19 Feb 2026** (sample data is generated for the full month—see `scripts/generate_sample_data_feb2026.py`).

---

# ACT 2: “Green Ops” Sustainability Dashboard

**One-line pitch:** “The platform isn’t just for developers—it’s for the C-Suite and ESG goals.”

## Story in one sentence

We combine **eBPF** (or CSP billing) usage per service and region with **carbon intensity** (grams CO2 per kWh by region). The dashboard shows that a fast service in a high-carbon region can cost more in carbon (and carbon credits) than a slightly slower one in a greener region.

## Data you need (ingest order)

| Order | File | Index name | Purpose |
|-------|------|------------|--------|
| 1 | `sample-data/carbon-intensity-by-region.csv` | `severn-vale-carbon-intensity` | Carbon grams per kWh (or kg) by region |
| 2 | `sample-data/service-usage-by-region-daily.ndjson` | `severn-vale-usage-by-region` | Daily CPU-hours (or kWh) and request_volume per service per region |
| 3 | (Optional) Reuse `sustainability-co2-daily.ndjson` | `severn-vale-sustainability` | Pre-aggregated emissions if you want a second view |

**Elastic ingest:** Same as above. Data views with **time field = `date`** (for daily series). Carbon intensity can be a **reference** index (region → g_co2_per_kwh) with or without a time field depending on how you upload.

## Scripted narrative (what to say)

1. **Setup**  
   *“We bring together infrastructure usage—from eBPF or cloud billing—and carbon intensity by region. That gives us a Green Ops view: not just cost, but environmental impact.”*

2. **Show the visual**  
   *“On the dashboard we have emissions—metric tons CO2 equivalent—and request volume by service and region. Service A is fast and handles a lot of traffic, but it runs in a region with high coal dependency. Service B runs in a greener region. So while A is ‘faster,’ it costs us more in carbon and in carbon credits.”*

3. **Value prop**  
   *“This isn’t just for developers. It’s for the C-Suite and ESG: one place to see how infrastructure choices affect both performance and sustainability.”*

## What to show (click path)

1. Open the **Green Ops** dashboard (or Discover with the usage + carbon indices).
2. Show **emissions_mt_co2e** (or **co2_kg** / 1000) by **service** and **region**.
3. Show **request_volume** (or CPU-hours) by **service** and **region**.
4. Point out: **high request_volume + high carbon intensity region** = high emissions; **similar volume + low carbon region** = lower emissions. Optionally compare “cost in carbon credits” if you add a simple calculated field or narrative.

## Key metrics for the storyline

- **emissions_mt_co2e** (or **co2_kg** / 1000) — from usage × carbon intensity or pre-aggregated sustainability data.
- **request_volume** — from eBPF-derived metrics or synthetic usage data.
- **carbon_intensity** (e.g. **g_co2_per_kwh**) — by region, for the “why” (coal vs renewable).

---

# End-to-end flow (recommended order for the demo)

1. **Before the demo:** Ingest all files; create data views; build the two dashboards (see below).
2. **Act 1:** SaaS-to-App Funnel — show Salesforce drop → OTel latency spike → “lost revenue” message.
3. **Act 2:** Green Ops — show emissions vs request volume by service/region → “C-Suite and ESG” message.
4. **Closing:** “One platform: OTel, eBPF, Salesforce, cost, carbon. Multiple sources, one place to correlate business and technical outcomes.”

---

# Dashboard and view build steps (Elastic)

## Data views to create

| Data view name | Indices | Time field |
|----------------|---------|------------|
| Severn Vale – Salesforce Funnel | `severn-vale-sf-funnel` | `date` |
| Severn Vale – OTel Summary | `severn-vale-otel-summary` | `date` |
| Severn Vale – Carbon Intensity | `severn-vale-carbon-intensity` | `effective_date` |
| Severn Vale – Usage by Region | `severn-vale-usage-by-region` | `date` |
| Severn Vale – Sustainability | `severn-vale-sustainability` | `date` |
| Severn Vale – Business metrics | `severn-vale-business-metrics` | `date` |
| Severn Vale – Salesforce pipeline | `severn-vale-salesforce-pipeline` | `date` |

## Dashboard 1: SaaS-to-App Conversion Funnel

Use **`ELASTIC-BUILD-STEPS.md` Step 3** for the full layout (recommended: **~8–10 panels** so it doesn’t look empty).

**Summary:**  
- **CRM row:** **trial_signups**, **conversion_rate_pct**, **leads**, **mql_count** (Salesforce Funnel data view).  
- **App row:** **p99_latency_ms**, **p50_latency_ms**, **request_count** for **account_creation** (OTel summary).  
- **Business row:** **value** by **date**, broken down by **metric** (business-metrics index), optionally split USD vs count/pct.  
- **Pipeline row:** stacked **pipeline_value_usd** by **stage** over **date** (salesforce-pipeline index).  
- Optional **Markdown** title strip at the top.

**Shared:** Time range **February 2026** (or zoom around **19 Feb**). One time picker for the whole dashboard.

## Dashboard 2: Green Ops Sustainability

- **Panel 1 – Bar or table:** Data view = Usage by Region. **emissions_mt_co2e** (or co2_kg/1000) by **service** and **region**. Title: “Emissions (mt CO2e) by service and region”.
- **Panel 2 – Bar or line:** **request_volume** by **service** and **region**. Title: “Request volume by service and region”.
- **Panel 3 (optional) – Table:** Carbon intensity by region (**region**, **g_co2_per_kwh**). Title: “Carbon intensity by region (g CO2/kWh)”.
- **Shared:** Time range **February 2026** (daily series in sample data are **1–28 Feb 2026**).

## Enrichment ideas (optional)

- **Calculated field:** In the usage-by-region data view, if you have **kwh** and **g_co2_per_kwh** in the same document (e.g. after a runtime field or ingest enrichment), add **emissions_kg = kwh * g_co2_per_kwh / 1000**.
- **Same dashboard:** Add a panel for **cost** (from your existing cloud cost index) so you show cost + carbon + volume in one place.
- **SLO (optional):** If you have real OTel traces, define an SLO on Account Creation latency and show SLO status on the funnel dashboard as another “why” signal.

---

# File checklist (all in `sample-data/`)

- [ ] `salesforce-lead-to-opportunity-daily.csv` — funnel + trial signups, conversion rate
- [ ] `otel-account-creation-latency-daily.ndjson` — P99/P50 by day for account_creation
- [ ] `carbon-intensity-by-region.csv` — region → g_co2_per_kwh
- [ ] `service-usage-by-region-daily.ndjson` — date, service, region, cpu_hours, request_volume, kwh, emissions_kg

After you upload these and create the data views and dashboards above, you can run the scripted storyline as-is.

---

# Quick reference: new files and indices (storyline)

| File | Index name | Used in |
|------|------------|--------|
| `sample-data/salesforce-lead-to-opportunity-daily.csv` | `severn-vale-sf-funnel` | Act 1 – SaaS funnel |
| `sample-data/otel-account-creation-latency-daily.ndjson` | `severn-vale-otel-summary` | Act 1 – OTel correlation |
| `sample-data/carbon-intensity-by-region.csv` | `severn-vale-carbon-intensity` | Act 2 – Green Ops (reference) |
| `sample-data/service-usage-by-region-daily.ndjson` | `severn-vale-usage-by-region` | Act 2 – Green Ops |

**Carbon intensity note:** `carbon-intensity-by-region.csv` includes **`effective_date`** (`2026-02-01`) as a regional snapshot. If your upload flow still needs a Kibana time field, map `effective_date` as **date** or duplicate a `date` column from that field.
