## Telemetry Management

The preferred approach of ingest and telemetry management is through OTel, of which we used the EDOT Collector with minor modifications.

#### Logstash
Initially the work was done with Logstash, and the config / pipeline configs are in the [Logstash folder](./Logstash/).
In order to prove that Logstash was redacting before it hit the network, Filebeat was configured to output to the console. See the [Filebeat folder](./Logstash/filebeat-console-debug/).

For the logstash pipeline, this can managed through Kibana:
- `**`Stack Management` → `Logstash` (or `Pipeline management` / `Ingest` → `Logstash Pipelines`, depending on your Kibana version).
- Click `Create pipeline` (or `Add pipeline`).

Both demos for PII and log severity were generated using a Cursor-built logging container, running in Docker. The fake logs were taken from an Elastic blog which contained a [python script that generated logs](https://github.com/bvader/elastic-pii/tree/main/python). Cursor took this python script and containerized it.
See: 
- [Running Logstash on Docker](https://www.elastic.co/docs/reference/logstash/docker)
- [How to remove PII from your Elastic data in 3 easy steps](https://www.elastic.co/observability-labs/blog/remove-pii-data)
- [Using NLP and Pattern Matching to Detect, Assess, and Redact PII in Logs -  Part 1](https://www.elastic.co/observability-labs/blog/pii-ner-regex-assess-redact-part-1)

The [dockerfile](./Logstash/docker-compose.yml) creates the containers for the Elastic Agent and Logstash. [Logstash.conf](./Logstash/logstash.conf) has the full configuration for PII redaction and filtering/output to S3.

#### EDOT / OTel

After building the above, the preferred way was to use EDOT, adding specific processors or exporters for redacting and outputting to S3.

Useful links:
- [Build a custom EDOT-like collector](https://www.elastic.co/docs/reference/edot-collector/custom-collector)
- [Components included in the EDOT Collector](https://www.elastic.co/docs/reference/edot-collector/components)

I believe that to add the necessary components you add them into this [manifest file](https://github.com/elastic/opentelemetry-collector-components/blob/main/distributions/elastic-components/manifest.yaml) as well as adding them into the respective [exporter], [processor](https://github.com/elastic/opentelemetry-collector-components/tree/main/processor) or [receiver](https://github.com/elastic/opentelemetry-collector-components/tree/main/receiver) folder.

This was done in Docker, but Cursor did it for me. The dockerfiles and everything needed should be in the [OTel folder](./OTel/edot-awss3/). We cloned the Elastic(?) version of the OTel demo to make it run on Docker,  and edited the [otel-collector config](./OTel/edot-awss3/opentelemetry-demo/src/otel-collector/). This is where the `s3exporter`, `filter` and `redact` processors were configured. The dockerfile to make the modified EDOT Collector is [here](./OTel/edot-awss3/Dockerfile) based on the [manifest additions](./OTel/edot-awss3/MANIFEST_ADDITION.txt).