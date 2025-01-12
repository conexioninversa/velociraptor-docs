---
title: LogScale.Events.Clients
hidden: true
tags: [Server Event Artifact]
---

This server side event monitoring artifact will watch a selection of client
monitoring artifacts for new events and push those to a LogScale (formerly
Humio) ingestion endpoint

NOTE: You must ensure you are collecting these artifacts from the
clients by adding them to the "Client Events" GUI.


```yaml
name: LogScale.Events.Clients
description: |
  This server side event monitoring artifact will watch a selection of client
  monitoring artifacts for new events and push those to a LogScale (formerly
  Humio) ingestion endpoint

  NOTE: You must ensure you are collecting these artifacts from the
  clients by adding them to the "Client Events" GUI.

type: SERVER_EVENT

parameters:
  - name: ingestApiBase
    description: API Base Url for LogScale server
    type: string
    default: https://cloud.community.humio.com/api
  - name: ingestToken
    description: Ingest token for API
    type: string
  - name: tagFields
    description: Comma-separated list of field names to use as tags in the message; Can be renamed with <oldname>=<newname>.
    default:
    type: string
  - name: numThreads
    description: Number of threads to start up to post events
    type: int
    default: 1
  - name: httpTimeout
    description: Timeout (in seconds) for http connection attempts
    type: int
    default: 10
  - name: batchingTimeoutMs
    description: Timeout (in ms) to batch events prior to sending
    type: int
    default: 30000
  - name: eventBatchSize
    description: Count of events to batch prior to sending
    type: int
    default: 2000
  - name: statsInterval
    description: Interval to post statistics to log (in seconds, 0 to disable)
    type: int
    default: 600
  - name: debug
    description: Enable verbose logging
    type: bool
    default: false
  - name: Artifacts
    type: artifactset
    artifact_type: CLIENT_EVENT
    description: Client artifacts to monitor

sources:
  - query: |
      LET artifacts_to_watch = SELECT Artifact FROM Artifacts
        WHERE log(message="Uploading artifact " + Artifact + " to LogScale")

      LET events = SELECT * FROM foreach(
          row=artifacts_to_watch,
          async=TRUE,   // Required for event queries in foreach()
          query={
             SELECT *, Artifact, timestamp(epoch=now()) AS timestamp
             FROM watch_monitoring(artifact=Artifact)
          })

      SELECT * FROM logscale_upload(
          query=events,
          apibaseurl=ingestApiBase,
          ingest_token=ingestToken,
          threads=numThreads,
          tag_fields=split(string=tagFields, sep=","),
          batching_timeout_ms=batchingTimeoutMs,
          event_batch_size=eventBatchSize,
          http_timeout=httpTimeout,
          debug=debug,
          stats_interval=statsInterval)

```
