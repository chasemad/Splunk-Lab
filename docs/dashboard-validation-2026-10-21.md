# Dashboard Studio validation — 2026-10-21

**Environment:** Splunk Cloud, `index="soc-live"`, monitored host `soc-ubuntu-01`.  
**Evidence:** Three screenshots supplied by the lab operator, each displaying the three-tab dashboard in view mode using **Last 24 hours**. Original image files are not yet part of the repository. All counts below are a point-in-time observation, **not continuing service-level measurements**.

## Visual/runtime checks

| Check | Observation | Status |
|---|---|---|
| SOC Overview | KPI panels, sourcetype chart, telemetry inventory, SSH sessions and file changes rendered | Rendering confirmed |
| Endpoint Activity | Sudo, execution, HTTP, file changes rendered with columns | Rendering confirmed |
| Investigation Workspace | Session correlation, cross-source timeline, file and process evidence rendered | Rendering confirmed |
| Report references | No missing-object banners visible in any of the three supplied screenshots | Apparent resolution, pending save/reopen check |
| Global time range | **Last 24 hours** input present on all tabs | Presence confirmed; propagation **not tested** |
| Overall telemetry KPI | 8,834 indexed events, one host, newest-event age displayed as 1 minute | Screenshot observation |

Source inventory counts from Overview: `syslog=5421`, `linux:audit=3104`, `linux_secure=303`, `soc:web:access=5`, and a manual HEC test source `=1`. These sum to **8,834**, independently corroborating the total-events KPI for the displayed window. Count agreement **does not** prove that all saved reports honor the time-range input.

## Findings requiring validation

### DV-001 — Saved report time ranges

Production JSON defaults `ds.search` data sources to the global `$global_time$` range; the eight saved reports use `ds.savedSearch`. Splunk's documentation differentiates saved-report execution and ad hoc search execution. Do **not** assume a change to the dashboard time picker affects the `ds.savedSearch` panels. Test **Last 24 hours → Last 15 minutes**, and record the KPI and at least one saved report table's behavior. If saved reports ignore the picker, fix the UX by explicitly distinguishing fixed-window report tables, or—after considering query cost—using appropriate `ds.search` sources for genuinely interactive investigation panels; keep canonical saved reports as independently executable detection artifacts. Do not add unsupported `ds.savedSearch` options without checking the installed Splunk build.

### DV-002 — HTTP diagnostic row classified as unparsed

The HTTP activity screenshot shows valid GET requests, including statuses 200 and 404, **plus one** `unparsed` row with server diagnostic text resembling `code 404, message File not found`. The current canonical SPL uses `coalesce(...,"unparsed")` to retain unmatched lines intentionally. A web-server status message should not be treated as a separate HTTP request. Consider separating request lines from diagnostic records, retaining malformed-log visibility in its own search or a separate data-quality count. Re-run and validate the revised search before editing the saved report in Splunk.

### DV-003 — Correlation scope and session attribution

The candidate correlation table displays source `172.16.110.128`, SSH PID `5124`, Auditd session `33`, `audited_execs=73`, and `monitored_file_ops=3`. The canonical `LINUX-COR-001` query aggregates **Auditd SYSCALL** records with the selected audit keys by `host,audit_session`, matches an Auditd LOGIN PID with the SSH authentication PID, and checks authentication/login timestamps are within 120 seconds. The 73 number is a count of selected execution SYSCALL records in that aggregation, **not necessarily 73 distinct user-entered commands**. Validate selected records, process content and effective time bounds before making attribution claims. Tighten correlation by session boundaries and host/boot context as appropriate if long search windows are used.

The visible **file-evidence table** shows earlier file operations under Auditd session `6`, whereas the correlated SSH session is `33`. They are different sessions: the dashboard's unfiltered evidence tables must **not** be interpreted as automatically belonging to the correlated SSH login. Future UX: add investigation controls for `audit_session` and/or associated case time window, or prominently label these panels as all recent endpoint activity rather than selected-session evidence.

### DV-004 — Presentation quality

Tables are comprehensive but some columns (`_raw`, multivalue `executed_processes`, UTC timestamps) crowd the full-screen view, particularly in the correlation and process evidence panels. Prefer a concise summary plus a clear raw-event drilldown, and explicitly label timezone and report time scope before final portfolio capture.

## Next validation steps

1. **Time picker A/B test**: screenshot Last 24 hours total and SSH / file evidence baseline, switch to Last 15 minutes, wait for searches to settle, record whether indexed-event total changes and whether historical report rows disappear. Do not infer a saved report is filtered just because the inline KPI changed.
2. **Validate source reports in Search & Reporting**: inspect exact SPL/time setting for each of the eight named saved searches; compare with the repository. Check sudo, process and HTTP fields and the five-event access-log content.
3. **Verify session 33 attribution**: compare the accepted SSH authentication PID and Auditd LOGIN PID/session/time, inspect audited execution/file `SYSCALL` records with `ses=33`, and compare the counts with `LINUX-COR-001`. Verify why the unfiltered evidence panels show `ses=6`.
4. **Resolve HTTP data-quality row**: edit and re-run candidate SPL to separate request entries from web-server diagnostics, then update the actual report and export aligned SPL.
5. **Save/reopen** the dashboard after functional corrections; export the actual Studio source, collect sanitized screenshots, and then perform the distinct SOC-003 exercise.

## Notes

These checks use observed screenshots and GitHub's *prepared* SPL. The saved report definitions currently in Splunk Cloud have **not yet been independently exported for byte-for-byte comparison**. The SOC-003 runbook remains unexecuted as a distinct case.
