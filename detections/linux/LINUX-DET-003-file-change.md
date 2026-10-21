# LINUX-DET-003 — Monitored Linux file changes

**Status:** Manual validation passed for three positive file changes and one read-only negative test; broader tuning, report saving and alert validation remain pending.
**Index:** `soc-live` (live Ubuntu audit telemetry)
**Purpose:** Surface successful audited changes in the intentionally monitored lab directory without double-counting multi-record Auditd events.

## What has actually been observed

An earlier exploratory SPL grouping query returned **25 matching raw log records grouped into 7 distinct audit event IDs** in a selected search window on 2026-10-21. A row for `/opt/soc-lab-test/soc002-test.txt` was associated with `ses=27`. Separately, SOC-002 confirmed the file's `openat` operation with `success=yes` and `nametype=CREATE`.

The updated candidate SPL has now been run against the live lab. On 2026-10-21, its search reported **34 matching input log records** and produced **10 grouped, successful file-change operations** over the selected time window. The grouped output included three operations against `/opt/soc-lab-test/det003-validation.txt` from Auditd session `33`. Do not confuse the number of input log records with the number of file operations; the query filters and groups audit records by event ID.

## Controlled validation — 2026-10-21

| Audit ID | Executing process | AArch64 syscall | PATH nametype | Result | Expected lab action |
|---|---|---|---|---|---|
| `1792583424.645:1038` | `bash` | `56` (`openat`) | `CREATE`, `PARENT` | `success=yes` | Create `det003-validation.txt` via shell redirection |
| `1792583424.649:1041` | `chmod` | `53` (`fchmodat`) | `NORMAL` | `success=yes` | Permission change |
| `1792583424.649:1042` | `bash` | `56` (`openat`) | `NORMAL`, `PARENT` | `success=yes` | Append to existing file via shell redirection |

Positive tests: **3/3** controlled write/attribute-change actions appeared as distinct grouped operations.

Negative test: the read-only `cat /opt/soc-lab-test/det003-validation.txt` execution appeared separately in the process audit stream at approximately 11:50:24 UTC as an `execve` event (session `33`, `euid=1000`), but **did not create a fourth matching file-change operation** in the grouped detection results. Other `cat` executions in the screenshot were earlier processes with different privilege context; those were not treated as additional file-change positives.

`PATH nametype=NORMAL` does not mean "benign"; it indicates how Auditd labels the referenced object. These tests cover only the configured watched lab directory and this specific behavior. Operational precision and alert reliability still need more tuning and baseline tests.

**Current stage:** Manual positive/negative validation complete; Splunk Report saving, noise tuning, scheduled alert testing and screenshot publication remain pending.

## Query

The canonical, copyable query is stored in [LINUX-DET-003-file-change.spl](LINUX-DET-003-file-change.spl).

~~~spl
index="soc-live" host="soc-ubuntu-01" source="/var/log/audit/audit.log"
("soc_lab_file_change" OR "/opt/soc-lab-test/")
| rex field=_raw "msg=audit\((?<audit_id>[^)]+)\)"
| rex field=_raw "type=(?<record_type>[A-Z_]+)"
| rex field=_raw "name=\"(?<file_path>/opt/soc-lab-test/[^\"]+)\""
| rex field=_raw "success=(?<operation_success>\w+)"
| rex field=_raw "comm=\"(?<process>[^\"]+)\""
| rex field=_raw "\bauid=(?<audit_user>\d+)"
| rex field=_raw "\bses=(?<audit_session>\d+)"
| eval has_file_rule=if(match(_raw,"key=\"soc_lab_file_change\""),1,0)
| stats earliest(_time) as first_seen
        max(has_file_rule) as has_file_rule
        values(file_path) as files
        values(operation_success) as outcomes
        values(process) as processes
        values(audit_user) as audit_users
        values(audit_session) as sessions
        values(record_type) as record_types
        count as raw_records
        by host audit_id
| where has_file_rule=1 AND isnotnull(files) AND mvfind(outcomes,"^yes$")>=0
| eval first_seen=strftime(first_seen,"%Y-%m-%d %H:%M:%S")
| table first_seen host audit_id files processes audit_users sessions outcomes raw_records record_types
| sort 0 -first_seen
~~~

## Why the SPL is written this way

- `SYSCALL`, `PATH`, `CWD`, and `PROCTITLE` are related records, not independent file actions.
- The `msg=audit(timestamp:serial)` identifier groups the records belonging to a single operation.
- The `has_file_rule` check ensures a file-change watch rule was involved, instead of relying on a pathname alone.
- `success=yes` filters to operations whose syscall reports success; the `PATH` record supplies the name, and another `SYSCALL` record supplies process and identity metadata.
- `raw_records` documents how many underlying log lines formed each grouped operation.

## Test and tuning plan

1. Run manually in Splunk with a time window covering the known SOC-002 test; confirm whether the current query returns its audit ID and associated file.
2. Generate a **controlled positive**: create a new file in the authorized test directory, inspect `PATH` and associated `SYSCALL`, and record the resulting audit ID.
3. Generate a second positive: modify or change permissions on an existing authorized test file; verify whether the scoped audit rule covers the behavior and whether the query finds it.
4. Generate a **controlled negative**: read a harmless file without changing it; this should not appear in a successful write/attribute-change detection.
5. Review noise from known lab operations. Compare the actor, executable, target path and session to a documented allowlist. Tune as narrowly as possible (for example, a known benign process *and* path), not by broadly excluding `socadmin` or all `touch` events.
6. Re-run with identical time windows before and after each tuning change; log true positives, expected benign activity, misses, and duplicate events.
7. Save the **tested** SPL as a Splunk **Report**: `LINUX-DET-003 - Monitored File Changes`. Use scheduled alerting only if the trial permits it and the logic, time window, volume and duplicate suppression have been tested.

### Important limitations

- Monitoring only `/opt/soc-lab-test` does **not** establish visibility into system-sensitive files such as `/etc/sudoers`.
- A successful filesystem syscall is not evidence of malicious intent.
- A file may be modified by many legitimate programs, and path-only alerting is noisy.
- These searches assume the rule is loaded, the forwarder is reading the audit log, and relevant records have not aged out of Splunk.
- For any production-style high-fidelity rule, first document the genuinely sensitive target paths, expected writers, and authorization baseline.

## Evidence placeholders

- [ ] Screenshot: exploratory search grouped 25 raw records into 7 audit operations.
- [ ] Screenshot: revised SPL results showing SOC-002 file creation.
- [x] Screenshot provided in conversation: positive create, chmod and append tests (repo image publication pending).
- [x] Screenshot provided in conversation: read-only cat process audit (repo image publication pending).
- [ ] Screenshot: before/after tuning comparison.
- [ ] Exported final saved report SPL and settings.
