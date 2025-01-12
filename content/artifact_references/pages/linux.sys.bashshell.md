---
title: Linux.Sys.BashShell
hidden: true
tags: [Client Artifact]
---

This artifact allows running arbitrary commands through the system
shell.

Since Velociraptor typically runs as root, the commands will also
run as root.

This is a very powerful artifact since it allows for arbitrary
command execution on the endpoints. Therefore this artifact requires
elevated permissions (specifically the `EXECVE`
permission). Typically it is only available with the `administrator`
role.


```yaml
name: Linux.Sys.BashShell
description: |
  This artifact allows running arbitrary commands through the system
  shell.

  Since Velociraptor typically runs as root, the commands will also
  run as root.

  This is a very powerful artifact since it allows for arbitrary
  command execution on the endpoints. Therefore this artifact requires
  elevated permissions (specifically the `EXECVE`
  permission). Typically it is only available with the `administrator`
  role.

required_permissions:
  - EXECVE

parameters:
  - name: Command
    default: "ls -l /"

sources:
  - query: |
      SELECT * FROM execve(argv=["/bin/bash", "-c", Command])

```
