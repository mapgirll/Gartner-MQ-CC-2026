# Severn Vale Digital — demo branding (single reference)

Use this fictional company consistently across narrative, indices, and Fleet.

| Item | Value |
|------|--------|
| **Company name** | Severn Vale Digital |
| **Backstory (optional one-liner)** | UK West Midlands–inspired digital / commerce platform (River Severn, Vale of Evesham, Malvern area). |
| **Fleet integration package name** | `severn_vale_digital` |
| **Typical custom data streams** | `commerce_kpis`, `batch_exports`, `app_latency_daily`, `sustainability_usage` |
| **Manual-upload index prefix** | `severn-vale-*` (e.g. `severn-vale-sf-funnel`, `severn-vale-otel-summary`) |
| **Kibana data view label pattern** | `Severn Vale – …` (e.g. Severn Vale – Salesforce Funnel) |
| **Logs data stream pattern (Fleet)** | `logs-severn_vale_digital.<dataset>-default` (verify in your stack) |

**Docs that use this branding:** `README-non-application-data.md`, `STORYLINE-scripted-demo.md`, `ELASTIC-BUILD-STEPS.md`, `CUSTOM-INTEGRATION-FORM.md`.

If you already ingested data into **`demo-gartner-*`** indices, either re-upload to **`severn-vale-*`** or keep old indices and only rename data views / narrative—indices and names do not need to match for the demo to work.

**Regenerate sample CSV/JSON:** from the Section1 folder run `python3 scripts/generate_sample_data_feb2026.py` (full **February 2026**; incident **19 Feb 2026**).
