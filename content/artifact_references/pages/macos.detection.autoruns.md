---
title: MacOS.Detection.Autoruns
hidden: true
tags: [Client Artifact]
---

This artifact collects evidence of autoruns. We also capture the files and upload them.

This code is based on
https://github.com/CrowdStrike/automactc/blob/master/modules/mod_autoruns_v102.py


```yaml
name: MacOS.Detection.Autoruns
description: |
   This artifact collects evidence of autoruns. We also capture the files and upload them.

   This code is based on
   https://github.com/CrowdStrike/automactc/blob/master/modules/mod_autoruns_v102.py

precondition: SELECT OS FROM info() WHERE OS =~ 'darwin'

parameters:
- name: sandboxed_loginitems
  default: /var/db/com.apple.xpc.launchd/disabled.*.plist

- name: cronTabGlob
  default: /private/var/at//tabs/*

- name: LaunchAgentsDaemonsGlob
  default: |
     ["/System/Library/LaunchAgents/*.plist","/Library/LaunchAgents/*.plist",
      "/Users/*/Library/LaunchAgents/*.plist","/private/var/*/Library/LaunchAgents/*.plist",
      "/System/Library/LaunchAgents/.*.plist","/Library/LaunchAgents/.*.plist",
      "/Users/*/Library/LaunchAgents/.*.plist", "/private/var/*/Library/LaunchAgents/.*.plist",
      "/System/Library/LaunchDaemons/*.plist","/Library/LaunchDaemons/*.plist",
      "/System/Library/LaunchDaemons/.*.plist","/Library/LaunchDaemons/.*.plist"]

- name: ScriptingAdditionsGlobs
  default: |
      ["/System/Library/ScriptingAdditions/*.osax","/Library/ScriptingAdditions/*.osax",
       "/System/Library/ScriptingAdditions/.*.osax","/Library/ScriptingAdditions/.*.osax"]

- name: StartupItemsGlobs
  default: |
       ["/System/Library/StartupItems/*/*","/Library/StartupItems/*/*"]

- name: MiscItemsGlobs
  default: |
      ["/private/etc/periodic.conf", "/private/etc/periodic/*/*", "/private/etc/*.local",
       "/private/etc/rc.common",
       "/private/etc/emond.d/*","/private/etc/emond.d/*/*"]

- name: LoginItemsGlobs
  default: |
      ["/Users/*/Library/Preferences/com.apple.loginitems.plist",
       "/private/var/*/Library/Preferences/com.apple.loginitems.plist"]

sources:
- name: Sandboxed Loginitems
  query: |
    SELECT OSPath,
           Mtime,
           plist(file=OSPath) AS Disabled,
           upload(file=OSPath) AS Upload
    FROM glob(globs=sandboxed_loginitems)

- name: crontabs
  query: |
    LET raw = SELECT * FROM foreach(
          row={
            SELECT OSPath, Name, Mtime,
                   upload(file=OSPath) AS Upload
            FROM glob(globs=split(string=cronTabGlob, sep=","))
          },
          query={
            SELECT OSPath, Name, Mtime, Upload,
              data, parse_string_with_regex(
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
                 "(?P<Command>.+)$"]) as Record

            /* Read lines from the file and filter ones that start with "#" */
            FROM split_records(
               filenames=OSPath,
               regex="\n", columns=["data"]) WHERE not data =~ "^\\s*#"
            }) WHERE Record.Command

    SELECT Record.Event AS Event,
               Mtime,
               Name AS User,
               Record.Minute AS Minute,
               Record.Hour AS Hour,
               Record.DayOfMonth AS DayOfMonth,
               Record.Month AS Month,
               Record.DayOfWeek AS DayOfWeek,
               Record.Command AS Command,
               OSPath AS Path,
               Upload
    FROM raw

- name: LaunchAgentsDaemons
  query: |

    LET launchd_config = SELECT OSPath, Mtime,
           plist(file=OSPath) AS LaunchdConfig,
           upload(file=OSPath) AS Upload
    FROM glob(globs=parse_json_array(data=LaunchAgentsDaemonsGlob))

    LET programs = SELECT OSPath, Mtime, LaunchdConfig,
           get(member="LaunchdConfig.Program",
               default=get(member="LaunchdConfig.ProgramArguments.0")) AS Program
    FROM launchd_config

    SELECT OSPath, Mtime, LaunchdConfig,
           Program, hash(path=Program) AS Hash,
           upload(file=OSPath) AS Upload
    FROM programs

- name: ScriptingAdditions
  query: |
    SELECT OSPath,
           Mtime,
           upload(file=OSPath) AS Upload
    FROM glob(globs=parse_json_array(data=ScriptingAdditionsGlobs))

- name: StartupItems
  query: |
    SELECT OSPath,
           Mtime,
           upload(file=OSPath) AS Upload
    FROM glob(globs=parse_json_array(data=StartupItemsGlobs))

- name: MiscItems
  query: |
    SELECT OSPath,
           Mtime,
           upload(file=OSPath) AS Upload
    FROM glob(globs=parse_json_array(data=MiscItemsGlobs))

- name: LoginItems
  query: |
    SELECT OSPath,
           Mtime,
           plist(file=OSPath) AS LoginItemConfig,
           upload(file=OSPath) AS Upload
    FROM glob(globs=parse_json_array(data=LoginItemsGlobs))

```
