## Setting up AWS Firehose integration in Elastic

*Don't use the Integrations > Firehose*
Easiest way is to use `Add Data` > `Cloud` > `AWS` > `AWS Firehose (Quickstart)`.

As of late 2025, setting the Parameters in the template to `elasticsearch.index` and `es_datastream_name` to the destination stream name (i.e. `logs`) seems to work to get it in the desired stream.