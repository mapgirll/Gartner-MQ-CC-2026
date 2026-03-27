# Geographic sample data (traffic + billing / orders)

Synthetic **daily aggregates by city** for **February 2026**, aligned with **Severn Vale Digital** and the **19 Feb 2026** incident.

## Files (under `sample-data/`)

| File | Format | Rows (approx.) |
|------|--------|----------------|
| `geographic-traffic-orders-daily.ndjson` | NDJSON | 28 days × 13 cities = **364** |
| `geographic-traffic-orders-daily.csv` | CSV | Same + header |

**Regenerate:** `python3 scripts/generate_sample_data_feb2026.py`

## What it represents

Each document is **one city × one day**:

- **`website_sessions`** — inferred site traffic attributed to metro area (IP / CDN edge region).
- **`lead_submissions`** — CRM lead forms tied to that geography.
- **`orders_placed`** + **`revenue_usd`** — orders with **billing address** in that city (and a small per-lead value rolled into revenue for demo totals).

**`market`:** `uk_home` (Severn Vale / UK core), `eu`, `na`, `apac` — used for filters or map styling.

**Coordinates:** Top-level **`latitude`** and **`longitude`** (numbers, WGS84). No nested `location` object — upload wizards often map nested JSON as **`object`**, which breaks Maps. CSV uses the same **`latitude`** / **`longitude`** column names.

## Cities (lat/lon are realistic centre points)

| City | Country | Role in demo |
|------|---------|----------------|
| Evesham, Malvern, Honeybourne, Worcester, Birmingham, London | UK | **Home market** — stronger **19 Feb** dip (correlates with funnel/latency story). |
| Dublin, Frankfurt, Amsterdam | IE / DE / NL | EU digital footprint. |
| New York, Seattle, Ashburn | US | NA billing / traffic (`NA` region in other datasets). |
| Singapore | SG | APAC touchpoint. |

## Incident behaviour (19 Feb 2026)

- **`uk_home`:** sessions / leads / orders scaled down ~**34–42%** vs a normal day.  
- **`eu` / `na` / `apac`:** smaller dip (~**6–15%**) so the map tells a “UK-weighted outage / friction” story without zeroing global cities.

## Elastic / Kibana

### Index mapping (recommended)

```json
"latitude": { "type": "double" },
"longitude": { "type": "double" }
```

### Steps

1. **Ingest** NDJSON or CSV → index e.g. **`severn-vale-geo-daily`**.  
2. **Data view:** time field **`date`**.  
3. **Elastic Maps:** add a layer and use **latitude and longitude in separate fields** (exact UI label varies by version):  
   - **Latitude:** `latitude`  
   - **Longitude:** `longitude`  
   - **Metric:** e.g. **Sum** of **`website_sessions`**, **`orders_placed`**, or **`revenue_usd`**.  
   - **Time:** February 2026; scrub to **19 Feb** for the UK home-market dip.  
4. **Dashboard:** Full-width **Maps** under the funnel charts works well.

**Optional:** If another integration needs a single **`geo_point`**, derive it in an ingest pipeline or runtime field from **`latitude`** / **`longitude`** — the sample NDJSON does **not** include a nested `location` object.

## Narration idea

*“Signups didn’t just disappear in the aggregate—our **geo** view shows the **home market around the Severn Vale and West Midlands** took the hit on the 19th, while US billing cities only softened slightly. That matches a **regionally routed** app stack or a **UK-heavy campaign** coupled to the slow **Account Creation** path.”*
