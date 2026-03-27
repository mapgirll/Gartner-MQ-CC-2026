## Cost, sustainability and business metrics

Demonstrate how the platform can be configured to ingest telemetry and events from multiple sources - including non-application data. Ensure you demonstrate, where you can:
a) OpenTelemetry
b) eBPF
**c) Cost, sustainability and business metrics**
d) events from other monitoring tools
e) other data - network observability, Cloud Service Provider and SaaS platform data
(such as SalesForce


For this section I combined showing cost, sustainability and business data with a custom integration.
In `Integrations` > `+ Create new integration` > `Create custom integration`.

Now, this doesn't actually import any data, it creates the schema and you set up the ingestion pipeline as if it was going to import custom data. I still showed the process and benefits of AI generating the correct schema and pipelines etc., *but* I did the manual upload process to get the data in: `Integrations` > `Upload a file`.

#### Custom data

The custom data was all created around a fictional company: **Severn Vale Digital**. 

All of the sample data was created for February 2026. There is a variety of data: application latency, business/salesforce data for pipelines, leads, funnel etc., sustainability data etc. This all comes together to make a unified overview dashboard that shows how pipelines/funnel is directly impacted by application/SaaS platofrm latency for signups. It has geospatial data for customers by country. It allows the tracking of carbon and other sustainabiltiy consumption.

There is a folder [`elastic-exports`](./Severn%20Vale%20Digital/elastic-exports/) that also contains a [full export](./Severn%20Vale%20Digital/elastic-exports/SevernVale-full-export.ndjson) of all Severn Vale Digital assets (dashboard, data, idle data stream) and an [export of the dashboard](./Severn%20Vale%20Digital/elastic-exports/SevernVale-dashboard.ndjson).