---
title: Server.Monitor.VeloMetrics
hidden: true
tags: [Server Artifact]
---

Get Velociraptor server metrics.


```yaml
name: Server.Monitor.VeloMetrics
description: |
  Get Velociraptor server metrics.

type: SERVER

parameters:
  - name: MetricsURL
    default: http://localhost:8003/metrics

sources:
  - query: |
        LET stats = SELECT parse_string_with_regex(string=Content,
           regex=[
             'client_comms_concurrency (?P<client_comms_concurrency>[^\\s]+)',
             'client_comms_current_connections (?P<client_comms_current_connections>[^\\s]+)',
             'flow_completion (?P<flow_completion>[^\\s]+)',
             'process_open_fds (?P<process_open_fds>[^\\s]+)',
             'uploaded_bytes (?P<uploaded_bytes>[^\\s]+)',
             'uploaded_files (?P<uploaded_files>[^\\s]+)',
             'stats_client_one_day_actives{version="[^"]+"} (?P<one_day_active>[^\\s]+)',
             'stats_client_seven_day_actives{version="[^"]+"} (?P<seven_day_active>[^\\s]+)'
           ]) AS Stat, {
              // On Windows Prometheus does not provide these so we get our own.
              SELECT Times.user + Times.system as CPU,
                     MemoryInfo.RSS as RSS
              FROM pslist(pid=getpid())
           } AS PslistStats
        FROM  http_client(url=MetricsURL, chunk_size=50000)

        SELECT now() AS Timestamp,
               PslistStats.RSS AS process_resident_memory_bytes,
               parse_float(string=Stat.client_comms_concurrency)
                      AS client_comms_concurrency,
               parse_float(string=Stat.client_comms_current_connections)
                      AS client_comms_current_connections,
               parse_float(string=Stat.flow_completion) AS flow_completion,
               parse_float(string=Stat.uploaded_bytes) AS uploaded_bytes,
               parse_float(string=Stat.uploaded_files) AS uploaded_files,
               parse_float(string=Stat.process_open_fds)
                     AS process_open_fds,
               PslistStats.CPU AS process_cpu_seconds_total,
               parse_float(string=Stat.one_day_active)
                     AS one_day_active,
               parse_float(string=Stat.seven_day_active)
                     AS seven_day_active
        FROM stats

```
