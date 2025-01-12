---
title: Linux.Sys.Crontab
hidden: true
tags: [Client Artifact]
---

Displays parsed information from crontab.


```yaml
name: Linux.Sys.Crontab
description: |
  Displays parsed information from crontab.
parameters:
  - name: cronTabGlob
    default: /etc/crontab,/etc/cron.d/**,/var/at/tabs/**,/var/spool/cron/**,/var/spool/cron/crontabs/**
  - name: cronTabScripts
    default: /etc/cron.daily/*,/etc/cron.hourly/*,/etc/cron.monthly/*,/etc/cron.weekly/*
  - name: Length
    default: 10000
    type: int

precondition: SELECT OS From info() where OS = 'linux'

sources:
  - name: CronTabs
    query: |
      LET raw = SELECT * FROM foreach(
          row={
            SELECT OSPath from glob(globs=split(string=cronTabGlob, sep=","))
          },
          query={
            SELECT OSPath, data, parse_string_with_regex(
              string=data,
              regex=[
                 /* Regex for event (Starts with @) */
                 "^(?P<Event>@[a-zA-Z]+)\\s+(?P<Command>.+)",

                 /* Regex for regular command. */
                 "^(?P<Minute>[^\\s]+)\\s+"+
                 "(?P<Hour>[^\\s]+)\\s+"+
                 "(?P<DayOfMonth>[^\\s]+)\\s+"+
                 "(?P<Month>[^\\s]+)\\s+"+
                 "(?P<DayOfWeek>[^\\s]+)\\s+"+
                 "(?P<User>[^\\s]+)\\s+"+
                 "(?P<Command>.+)$"]) as Record

            /* Read lines from the file and filter ones that start with "#" */
            FROM split_records(
               filenames=OSPath,
               regex="\n", columns=["data"]) WHERE not data =~ "^\\s*#"
            }) WHERE Record.Command

      SELECT Record.Event AS Event,
               Record.User AS User,
               Record.Minute AS Minute,
               Record.Hour AS Hour,
               Record.DayOfMonth AS DayOfMonth,
               Record.Month AS Month,
               Record.DayOfWeek AS DayOfWeek,
               Record.Command AS Command,
               OSPath AS Path
      FROM raw
  - name: CronScripts
    query: |
      SELECT Mtime, OSPath, read_file(filename=OSPath,length=Length) AS Content
      FROM glob(globs=split(string=cronTabScripts, sep=","))
  - name: Uploaded
    query: |
      SELECT OSPath, upload(file=OSPath) AS Upload
      FROM glob(globs=split(string=cronTabGlob + "," + cronTabScripts, sep=","))

```
