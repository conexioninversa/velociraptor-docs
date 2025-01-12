---
title: Linux.Applications.Docker.Info
hidden: true
tags: [Client Artifact]
---

Get Dockers info by connecting to its socket.

```yaml
name: Linux.Applications.Docker.Info
description: Get Dockers info by connecting to its socket.
parameters:
  - name: dockerSocket
    description: |
      Docker server socket. You will normally need to be root to connect.
    default: /var/run/docker.sock
sources:
  - precondition: |
      SELECT OS From info() where OS = 'linux'
    query: |
        LET data = SELECT parse_json(data=Content) as JSON
        FROM http_client(url=dockerSocket + ":unix/info")

        SELECT JSON.ID as ID,
               JSON.Containers as Containers,
               JSON.ContainersRunning as ContainersRunning,
               JSON.ContainersPaused as ContainersPaused,
               JSON.ContainersStopped as ContainersStopped,
               JSON.Images as Images,
               JSON.Driver as Driver,
               JSON.MemoryLimit as MemoryLimit,
               JSON.SwapLimit as SwapLimit,
               JSON.KernelMemory as KernelMemory,
               JSON.CpuCfsPeriod as CpuCfsPeriod,
               JSON.CpuCfsQuota as CpuCfsQuota,
               JSON.CPUShares as CPUShares,
               JSON.CPUSet as CPUSet,
               JSON.IPv4Forwarding as IPv4Forwarding,
               JSON.BridgeNfIptables as BridgeNfIptables,
               JSON.BridgeNfIp6tables as BridgeNfIp6tables,
               JSON.OomKillDisable as OomKillDisable,
               JSON.LoggingDriver as LoggingDriver,
               JSON.CgroupDriver as CgroupDriver,
               JSON.KernelVersion as KernelVersion,
               JSON.OperatingSystem as OperatingSystem,
               JSON.OSType as OSType,
               JSON.Architecture as Architecture,
               JSON.NCPU as NCPU,
               JSON.MemTotal as MemTotal,
               JSON.HttpProxy as HttpProxy,
               JSON.HttpsProxy as HttpsProxy,
               JSON.NoProxy as NoProxy,
               JSON.Name as Name,
               JSON.ServerVersion as ServerVersion,
               JSON.DockerRootDir as DockerRootDir
        FROM data

```
