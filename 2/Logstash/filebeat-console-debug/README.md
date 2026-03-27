# Filebeat console output (redaction proof)

This runs **Filebeat** with the same filestream input and PII redaction processors as your Elastic Agent, but **outputs only to the console** (stdout). No data is sent to Elasticsearch from this container.

Use it to prove that redaction happens **inside the beat** (before any output), not in Elasticsearch.

## Run and watch

```bash
# From logstash/
docker compose up -d log-generator-logstash   # ensure logs are being written
docker compose up filebeat-redaction-proof   # foreground so you see stdout
```

Or run in background and follow logs:

```bash
docker compose up -d filebeat-redaction-proof
docker logs -f filebeat-redaction-proof
```

You should see JSON events with `userEmail` → `[REDACTED_EMAIL]`, `clientIp` → `[REDACTED_IP]`, and any email/IP in `message` redacted the same way.

## Config

- **Input:** Same paths as the agent: `/var/log/app.log`, `application.log`, `service.log` (from shared `logdata` volume).
- **Parsers:** `ndjson` with `message_key: message`.
- **Processors:** Same four `replace` processors as `telemetry-mgmt/filestream-processors.yaml`.
- **Output:** `output.console` with `pretty: true` only (no Elasticsearch).

Elastic Agent under Fleet cannot add a console output for its data; this standalone Filebeat container is the way to inspect the post-processor (redacted) stream.
