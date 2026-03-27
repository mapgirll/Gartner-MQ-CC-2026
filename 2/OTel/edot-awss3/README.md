# EDOT Collector with AWS S3 Exporter, Filter Processor, and Redaction Processor

Custom image based on the **Elastic Distribution of OpenTelemetry (EDOT) Collector** that adds:

- **exporter/awss3exporter** — [opentelemetry-collector-contrib/exporter/awss3exporter](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/awss3exporter) for writing telemetry (e.g. OTLP JSON) to AWS S3.
- **processor/filterprocessor** — Already included in the [elastic-components](https://github.com/elastic/opentelemetry-collector-components/tree/main/distributions/elastic-components) manifest; used to include/exclude spans, metrics, or logs by pattern.
- **processor/redactionprocessor** — [opentelemetry-collector-contrib/processor/redactionprocessor](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor/redactionprocessor) to remove or mask sensitive attributes (PII, tokens, etc.) before export.

EDOT reference:

- [Default configuration of the EDOT Collector (standalone)](https://www.elastic.co/docs/reference/edot-collector/config/default-config-standalone)
- [Quickstart for Docker on Elastic Cloud Hosted](https://www.elastic.co/docs/solutions/observability/get-started/opentelemetry/quickstart/ech/docker)

The standard EDOT Docker image (`elastic/elastic-agent:9.3.1` with `COLLECTOR_CONTRIB_IMAGE`) does not include the contrib `awss3exporter` or `redactionprocessor`. This project builds a custom collector from [elastic/opentelemetry-collector-components](https://github.com/elastic/opentelemetry-collector-components) with the same EDOT components **plus** `awss3exporter` and `redactionprocessor` (and keeps `filterprocessor`), then packages it in a minimal Docker image.

---

## What’s in the manifest

- **manifest.yaml** — Copy of the [elastic-components manifest](https://github.com/elastic/opentelemetry-collector-components/blob/main/distributions/elastic-components/manifest.yaml) with:
  - **Extra exporter:** `github.com/open-telemetry/opentelemetry-collector-contrib/exporter/awss3exporter v0.147.0`
  - **Extra processor:** `github.com/open-telemetry/opentelemetry-collector-contrib/processor/redactionprocessor v0.147.0`  
  The **filterprocessor** is already present in the upstream manifest; no change needed there.

The build uses the OpenTelemetry Collector Builder (OCB) inside a clone of `elastic/opentelemetry-collector-components`, with this manifest replacing `distributions/elastic-components/manifest.yaml`.

---

## Build the Docker image

From the **edot-awss3** directory (so `manifest.yaml` is in the build context):

```bash
cd edot-awss3
docker build -t edot-awss3 .
```

Build args (optional):

- `GO_VERSION` — Go version for the builder stage (default: `1.23`).
- `ELASTIC_OC_VERSION` — Branch/tag of [elastic/opentelemetry-collector-components](https://github.com/elastic/opentelemetry-collector-components) (default: `main`). Use a tag (e.g. `v0.20.0`) for a reproducible build.

Example with tag:

```bash
docker build --build-arg ELASTIC_OC_VERSION=v0.20.0 -t edot-awss3:v0.20.0 .
```

---

## Run the collector

Use your own config that wires OTLP → processors (e.g. `filterprocessor`) → Elasticsearch and/or `awss3`:

```bash
docker run -d \
  --name edot-collector \
  -p 4317:4317 -p 4318:4318 \
  -v $(pwd)/config.example.yaml:/etc/otelcol-config/config.yaml:ro \
  -e ELASTIC_ENDPOINT=https://your-deployment.es.us-central1.gcp.cloud.es.io:443 \
  -e ELASTIC_API_KEY=your-api-key \
  -e AWS_REGION=us-east-1 \
  -e S3_BUCKET=your-otel-bucket \
  -e S3_PREFIX=otel \
  edot-awss3 \
  --config=/etc/otelcol-config/config.yaml
```

Or use the example Compose file (set env vars in `.env` or the shell):

```bash
cp config.example.yaml config.yaml   # edit as needed
docker compose -f compose.example.yml up -d
```

---

## Example config

**config.example.yaml** shows:

- **Receivers:** OTLP gRPC (4317) and HTTP (4318).
- **Processors:** `memory_limiter`, `batch`, `filter/traces` (filterprocessor), and `redaction` (redactionprocessor) for traces/metrics/logs.
- **Exporters:**
  - **elasticsearch/otel** — EDOT-style export to Elasticsearch (use `ELASTIC_ENDPOINT` and `ELASTIC_API_KEY`).
  - **awss3** — Archive to S3 (use `AWS_REGION`, `S3_BUCKET`, optional `S3_PREFIX`; AWS credentials via env/instance role).

You can add or change filter rules, redaction (allowed/blocked keys and values), and pipelines as needed; see [filterprocessor](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor/filterprocessor), [redactionprocessor](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor/redactionprocessor), and [awss3exporter](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/awss3exporter) docs.

---

## Building without Docker (local)

If you prefer to build the binary locally, follow the [official Elastic instructions](https://github.com/elastic/opentelemetry-collector-components#building-a-collector-with-a-custom-component):

1. Clone the Elastic components repo:
   ```bash
   git clone https://github.com/elastic/opentelemetry-collector-components.git
   cd opentelemetry-collector-components
   ```
2. Replace the distribution manifest with the one from this project (or add the `awss3exporter` and `redactionprocessor` lines per `MANIFEST_ADDITION.txt`):
   ```bash
   cp /path/to/edot-awss3/manifest.yaml distributions/elastic-components/manifest.yaml
   ```
3. Build using the repo’s Makefile (uses the builder from `internal/tools`; requires Go 1.25+):
   ```bash
   TARGET_GOOS=linux make genelasticcol   # optional: CGO_ENABLED=0 for static binary
   ```
4. Binary path: `_build/elastic-collector-components`. To build a Docker image: `make builddocker TAG=v0.1.0`.

---

## Summary

| Item | Description |
|------|-------------|
| **Base** | EDOT (Elastic Distribution of OpenTelemetry) via [elastic/opentelemetry-collector-components](https://github.com/elastic/opentelemetry-collector-components) |
| **Added** | **awss3exporter** (contrib) for S3 export; **redactionprocessor** (contrib) for PII/sensitive redaction |
| **Already present** | **filterprocessor** (contrib) in elastic-components manifest |
| **Build** | Docker build from `edot-awss3` dir, or local OCB with manifest replace |
| **Run** | Same as EDOT: mount config, set env vars (Elastic + optional AWS), expose 4317/4318 |
