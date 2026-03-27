# Custom integration form – copy-paste values (Elastic)

Use this to fill the **custom integration** wizard (package name, data stream, collection method, log formats). Names must be **≥2 characters**, **start with a letter**, and use only **lowercase letters, numbers, underscores** (`a-z`, `0-9`, `_`).

**Branding cheat sheet:** **`SEVERN-VALE-DIGITAL.md`**.

---

## Fictional company for this demo

**Severn Vale Digital** — fictional UK West Midlands–inspired digital / commerce business (River Severn, Vale of Evesham, Malvern area). Use this name in titles and narrative; use the **integration package name** below in Fleet and pipelines.

| What | Value |
|------|--------|
| **Company (display)** | Severn Vale Digital |
| **Integration package name** | `severn_vale_digital` |

---

## Recommendation: don’t use `test` as the package name

`test` is valid, but it’s generic in pipelines, data stream names, and Fleet UI. Use a **company-scoped** name like **`severn_vale_digital`** so **Ingest Pipelines** and **Data streams** read clearly in the UI.

---

## Option A – HTTP Endpoint (best for “SaaS / API / webhook” story)

**When to use:** You want to show **push** ingestion: a script, Postman, or a mock “CRM export job” POSTs **JSON/NDJSON** to the Agent. Strong narrative for *“events from other tools / APIs.”*

| Field | Suggested value (Severn Vale Digital) |
|-------|----------------------------------------|
| **Integration package name** | `severn_vale_digital` |
| **Data stream title** | `Severn Vale Digital – commerce KPIs` |
| **Data stream description** | `Daily funnel and conversion metrics from CRM and product analytics, delivered via HTTP (non-application business data).` |
| **Data stream name** | `commerce_kpis` |
| **Data collection method** | **HTTP Endpoint** |

**Typical resulting data stream pattern** (verify in your UI after creation):

`logs-severn_vale_digital.commerce_kpis-default`

*(Exact suffix may vary by stack version; check **Stack Management → Index Management** or the integration summary.)*

**Logs – supported formats** (select what your wizard offers; align with what you actually send):

| Format | Use in this demo |
|--------|------------------|
| **JSON** | Single JSON object per request body, or array of objects (if supported). |
| **NDJSON / newline-delimited JSON** | One JSON object per line (matches `otel-account-creation-latency-daily.ndjson` and `service-usage-by-region-daily.ndjson` style). |

If the form only allows one primary format, choose **JSON** and send **one object per POST**, or **NDJSON** if the endpoint accepts raw body with multiple lines (per Elastic docs for that input).

**Sample POST body (single JSON document):**

```json
{"date": "2026-02-19", "source": "salesforce", "trial_signups": 116, "conversion_rate_pct": 2.43}
```

---

## Option B – Filestream (best for “batch files / exports on disk” story)

**When to use:** You want to show **file-based** collection: Agent reads a directory where **billing, CRM exports, or carbon CSVs** land (or you append NDJSON lines). Strong narrative for *“scheduled exports from CSP / CRM / internal tools.”*

| Field | Suggested value (Severn Vale Digital) |
|-------|----------------------------------------|
| **Integration package name** | `severn_vale_digital` |
| **Data stream title** | `Severn Vale Digital – batch exports` |
| **Data stream description** | `JSON/NDJSON exports from FinOps, CRM, and sustainability jobs collected from disk.` |
| **Data stream name** | `batch_exports` |
| **Data collection method** | **Filestream** |

**Path to configure on the host running Elastic Agent:** e.g. `/var/severn_vale/exports/*.ndjson` or a single file you rotate during the demo.

**Logs – supported formats:**

| Format | Use in this demo |
|--------|------------------|
| **JSON** | If each line or file is a JSON object. |
| **NDJSON** | Preferred for line-by-line appends (same shape as your `sample-data/*.ndjson` files). |

**Note:** Raw **CSV** is often not auto-parsed as structured JSON; for Filestream + ECS mapping, prefer **NDJSON** or **JSON** (one document per line). You can still demo CSV by converting a small export to NDJSON once, or by using a separate pipeline/parser if your version supports it.

---

## Quick decision: HTTP Endpoint vs Filestream

| Criterion | HTTP Endpoint | Filestream |
|-----------|----------------|------------|
| **Demo narrative** | “SaaS / API pushes events to Elastic.” | “Exports land as files; Agent tails them.” |
| **Live demo ease** | `curl` or Postman during the session. | Pre-place files or `echo >> file` on the Agent host. |
| **Firewall / security** | Needs reachable listener on Agent (or proxy). | No inbound HTTP; only local disk access. |
| **Buyer story** | Other tools / webhooks / SaaS. | CSP/CRM **exports**, batch jobs, shared folders. |

**Practical pick for a conference demo:** **HTTP Endpoint** if you can hit the URL from your laptop; **Filestream** if inbound HTTP is hard and you control the Agent machine.

---

## If you need a second data stream (same package)

Some wizards let you add another stream after the first. Examples (still under **`severn_vale_digital`**):

| Data stream name | Title | Collection | Purpose |
|------------------|-------|------------|--------|
| `app_latency_daily` | Severn Vale Digital – app latency rollups | HTTP or Filestream | Synthetic OTel-style summary (`p99_latency_ms`, `service_name`). |
| `sustainability_usage` | Severn Vale Digital – sustainability usage | HTTP or Filestream | `emissions_mt_co2e`, `request_volume`, `region`. |

Keep **one package name** and use **different data stream names** so pipelines stay clear.

---

## ECS / pipeline note

After you upload sample logs in the wizard, Elastic uses them to suggest **ECS** mappings. Prefer field names that map cleanly when you can, e.g.:

- `@timestamp` or a `date` field the wizard maps to **`@timestamp`**
- `message` for raw line (optional)
- `event.dataset` / `service.name` if you add them in JSON for clearer Observability views

Your demo can still use custom fields (`trial_signups`, `emissions_mt_co2e`); the pipeline step is mainly to normalize types and timestamps.

---

## Checklist before clicking Create

- [ ] Package name: lowercase, starts with letter, only `a-z`, `0-9`, `_` (e.g. `severn_vale_digital`).
- [ ] Data stream name: same rules (e.g. `commerce_kpis` or `batch_exports`).
- [ ] Title/description: human-readable; won’t affect pipeline syntax.
- [ ] Collection method matches how you’ll demo (HTTP vs files).
- [ ] Log format(s) match what you send: **JSON** and/or **NDJSON** per [Elastic documentation for your integration type](https://www.elastic.co/docs).

---

## Link to your storyline data

Sample bodies/files that match this integration:

- `sample-data/salesforce-lead-to-opportunity-daily.csv` → convert line(s) to JSON/NDJSON for HTTP/Filestream, or use NDJSON you generate from the CSV.
- `sample-data/otel-account-creation-latency-daily.ndjson` → send as NDJSON (Filestream) or one line per POST (HTTP).
- `sample-data/service-usage-by-region-daily.ndjson` → same as above.

After the integration is created, add a **data view** on the new `logs-severn_vale_digital.*` pattern (or the exact data stream name shown in Fleet) and add panels to your dashboards.
