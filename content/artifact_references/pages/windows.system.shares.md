---
title: Windows.System.Shares
hidden: true
tags: [Client Artifact]
---

This artifact will extract network shares per machine.


```yaml
name: Windows.System.Shares
author: 'Matt Green - @mgreen27'
description: |
   This artifact will extract network shares per machine.

type: CLIENT

parameters:
  - name: NameRegex
    description: Regex filter for share name. e.g Admin\$ for Admin$
    default: .
    type: regex
  - name: PathRegex
    description: Regex filter for local path. e.g C:\\Windows$ for Admin$
    default: .
    type: regex

sources:
  - precondition:
      SELECT OS From info() where OS = 'windows'

    query: |
        SELECT Name, Path, Caption, Status,MaximumAllowed,AllowMaximum,InstallDate
        FROM wmi(query='SELECT * FROM Win32_Share',namespace='root/cimv2')
        WHERE Name =~ NameRegex AND Path =~ PathRegex
```
