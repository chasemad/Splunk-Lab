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

## Time picker A/B test — operator screenshots, 2026-10-21

**Executed.** The operator changed the global time picker from **Last 24 hours** to **Last 15 minutes** and shared screenshots of all three tabs.

- **Inline KPIs:** indexed event count fell from **8,834** to **36**; observed hosts remained at 1, and the event-volume chart updated to the short period. The inline search token wiring is functioning.
- **Saved-report panels:** the Endpoint Activity sudo table still displayed older `auditctl -l` and forwarder restart commands; the web table retained early-day GET and diagnostic entries; the file-change table still displayed earlier session 6 operations. Investigation Workspace still displayed the previous correlation (`ssh_pid=5124`, `ses=33`) and earlier timeline rows. This demonstrates the current saved-report panels **do not respond to the global time picker** despite the input visibly changing.
- **Scope:** this establishes a functional dashboard issue, not lost telemetry or broken report references.

### Recommended low-risk remediation test

Splunk's 2026 Dashboard Studio [Call saved searches](https://help.splunk.com/en/splunk-enterprise/create-dashboards-and-reports/dashboard-studio/10.6/use-data-sources/call-saved-searches) documentation specifically describes a **saved-search time range** selection in the UI with three modes: *Inherit the time range from the saved search*, *Any time range input on the page*, or *static time range*. Because we have not inspected the installed version's **generated source representation** for this setting, **do not invent undocumented `ds.savedSearch` JSON properties yet**. Splunk's source documentation lists `ref`, `app`, and unscheduled refresh properties for this source type.

First test the setting on the separately created **SOC Lab | Dashboard Studio Diagnostic (Temporary)** dashboard:
1. Open in Edit, select the saved file-change report table on the right and edit its data source (or use Data source overview).
2. Choose `Any time range input on the page` and bind to its existing `Diagnostic time range` input if the UI offers it.
3. Apply and test **Last 15 minutes** (historical rows should disappear) versus **Last 24 hours** (historical rows return). Screenshot the UI source setting and resulting tables.
4. Inspect the dashboard Source editor **after UI Apply** to learn the actual schema supported by this Splunk Cloud instance.
5. If successful, update the eight production saved sources consistently using the native UI and export the working production source to GitHub. If the option is missing or still ignored, compare a bounded `ds.search`-based diagnostic using the canonical saved SPL before changing production.

The production dashboard is already working except for global saved-report time filtering and the other separately documented data-quality/attribution issues. **Avoid replacing it with an untested conversion.**

## Data source editor configuration observed — 2026-10-21

Operator screenshots of the **production dashboard** show the `LINUX-MON-001 — Telemetry Health` data source editor:

- Source name field is blank and the UI displays **Please enter a data source name**. The Data source overview consequently lists saved sources as **Unnamed**.
- The selected time-range mode is **Inherit from saved search**, displaying the saved report's **Last 24 hours** window. This is direct evidence of why the telemetry saved-report panel does not respond to the dashboard input currently.
- Global time-range input is titled **Investigation time range**, token **`global_time`**, default **Last 24 hours**.
- The saved report shows app `search`, owner `sc_admin`, permissions `Private`. Its displayed SPL matches the canonical telemetry inventory query.
- Source editor shows dashboard defaults linking `ds.search` to `$global_time.earliest# Dashboard Studio validation — 2026-10-21

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

## Time picker A/B test — operator screenshots, 2026-10-21

**Executed.** The operator changed the global time picker from **Last 24 hours** to **Last 15 minutes** and shared screenshots of all three tabs.

- **Inline KPIs:** indexed event count fell from **8,834** to **36**; observed hosts remained at 1, and the event-volume chart updated to the short period. The inline search token wiring is functioning.
- **Saved-report panels:** the Endpoint Activity sudo table still displayed older `auditctl -l` and forwarder restart commands; the web table retained early-day GET and diagnostic entries; the file-change table still displayed earlier session 6 operations. Investigation Workspace still displayed the previous correlation (`ssh_pid=5124`, `ses=33`) and earlier timeline rows. This demonstrates the current saved-report panels **do not respond to the global time picker** despite the input visibly changing.
- **Scope:** this establishes a functional dashboard issue, not lost telemetry or broken report references.

### Recommended low-risk remediation test

Splunk's 2026 Dashboard Studio [Call saved searches](https://help.splunk.com/en/splunk-enterprise/create-dashboards-and-reports/dashboard-studio/10.6/use-data-sources/call-saved-searches) documentation specifically describes a **saved-search time range** selection in the UI with three modes: *Inherit the time range from the saved search*, *Any time range input on the page*, or *static time range*. Because we have not inspected the installed version's **generated source representation** for this setting, **do not invent undocumented `ds.savedSearch` JSON properties yet**. Splunk's source documentation lists `ref`, `app`, and unscheduled refresh properties for this source type.

First test the setting on the separately created **SOC Lab | Dashboard Studio Diagnostic (Temporary)** dashboard:
1. Open in Edit, select the saved file-change report table on the right and edit its data source (or use Data source overview).
2. Choose `Any time range input on the page` and bind to its existing `Diagnostic time range` input if the UI offers it.
3. Apply and test **Last 15 minutes** (historical rows should disappear) versus **Last 24 hours** (historical rows return). Screenshot the UI source setting and resulting tables.
4. Inspect the dashboard Source editor **after UI Apply** to learn the actual schema supported by this Splunk Cloud instance.
5. If successful, update the eight production saved sources consistently using the native UI and export the working production source to GitHub. If the option is missing or still ignored, compare a bounded `ds.search`-based diagnostic using the canonical saved SPL before changing production.

The production dashboard is already working except for global saved-report time filtering and the other separately documented data-quality/attribution issues. **Avoid replacing it with an untested conversion.**

 and `$global_time.latest# Dashboard Studio validation — 2026-10-21

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

## Time picker A/B test — operator screenshots, 2026-10-21

**Executed.** The operator changed the global time picker from **Last 24 hours** to **Last 15 minutes** and shared screenshots of all three tabs.

- **Inline KPIs:** indexed event count fell from **8,834** to **36**; observed hosts remained at 1, and the event-volume chart updated to the short period. The inline search token wiring is functioning.
- **Saved-report panels:** the Endpoint Activity sudo table still displayed older `auditctl -l` and forwarder restart commands; the web table retained early-day GET and diagnostic entries; the file-change table still displayed earlier session 6 operations. Investigation Workspace still displayed the previous correlation (`ssh_pid=5124`, `ses=33`) and earlier timeline rows. This demonstrates the current saved-report panels **do not respond to the global time picker** despite the input visibly changing.
- **Scope:** this establishes a functional dashboard issue, not lost telemetry or broken report references.

### Recommended low-risk remediation test

Splunk's 2026 Dashboard Studio [Call saved searches](https://help.splunk.com/en/splunk-enterprise/create-dashboards-and-reports/dashboard-studio/10.6/use-data-sources/call-saved-searches) documentation specifically describes a **saved-search time range** selection in the UI with three modes: *Inherit the time range from the saved search*, *Any time range input on the page*, or *static time range*. Because we have not inspected the installed version's **generated source representation** for this setting, **do not invent undocumented `ds.savedSearch` JSON properties yet**. Splunk's source documentation lists `ref`, `app`, and unscheduled refresh properties for this source type.

First test the setting on the separately created **SOC Lab | Dashboard Studio Diagnostic (Temporary)** dashboard:
1. Open in Edit, select the saved file-change report table on the right and edit its data source (or use Data source overview).
2. Choose `Any time range input on the page` and bind to its existing `Diagnostic time range` input if the UI offers it.
3. Apply and test **Last 15 minutes** (historical rows should disappear) versus **Last 24 hours** (historical rows return). Screenshot the UI source setting and resulting tables.
4. Inspect the dashboard Source editor **after UI Apply** to learn the actual schema supported by this Splunk Cloud instance.
5. If successful, update the eight production saved sources consistently using the native UI and export the working production source to GitHub. If the option is missing or still ignored, compare a bounded `ds.search`-based diagnostic using the canonical saved SPL before changing production.

The production dashboard is already working except for global saved-report time filtering and the other separately documented data-quality/attribution issues. **Avoid replacing it with an untested conversion.**

, and additionally displays a `ds.spl` default in the live dashboard. Those defaults alone have **not** made the existing saved report sources respond.

**Recommended next action:** Edit *this one* saved source in the production dashboard: give it a clear UI name (e.g. `Telemetry Health`) and set its Time range dropdown to **Any time range input on the page**, selecting **Investigation time range** if asked. Click **Apply and close**, then test the rendered telemetry table with **Last 15 minutes** and **Last 24 hours** before changing the other seven sources or committing regenerated JSON. This follows [Splunk's native saved search editor guidance](https://help.splunk.com/en/splunk-cloud-platform/create-dashboards-and-reports/dashboard-studio/10.6/use-data-sources/call-saved-searches). The UI may write a special source representation; inspect/export the actual dashboard source after confirming the setting works. Do **not** infer that a browser Find result of `0/0` in Splunk's code editor proves a source ID is absent.

## Next validation steps

1. **Saved report time control pilot**: test the native time-range selection in the temporary diagnostic dashboard before applying it to the eight live production report sources. The global time picker A/B test has confirmed saved sources currently ignore it.
2. **Validate source reports in Search & Reporting**: inspect exact SPL/time setting for each of the eight named saved searches; compare with the repository. Check sudo, process and HTTP fields and the five-event access-log content.
3. **Verify session 33 attribution**: compare the accepted SSH authentication PID and Auditd LOGIN PID/session/time, inspect audited execution/file `SYSCALL` records with `ses=33`, and compare the counts with `LINUX-COR-001`. Verify why the unfiltered evidence panels show `ses=6`.
4. **Resolve HTTP data-quality row**: edit and re-run candidate SPL to separate request entries from web-server diagnostics, then update the actual report and export aligned SPL.
5. **Save/reopen** the dashboard after functional corrections; export the actual Studio source, collect sanitized screenshots, and then perform the distinct SOC-003 exercise.

## Notes

These checks use observed screenshots and GitHub's *prepared* SPL. The saved report definitions currently in Splunk Cloud have **not yet been independently exported for byte-for-byte comparison**. The SOC-003 runbook remains unexecuted as a distinct case.

## Bidirectional saved-report filter test — 2026-10-21

The operator shared two successive **Endpoint Activity** screenshots after configuring the saved-report time inputs.

- **Last 15 minutes:** the Sudo Activity table and Selected Process Executions table display `No search results returned`. Hovering over the yellow indicator on the process panel confirms the message is `No search results returned`, not a query syntax error.
- **Last 24 hours:** historical sudo `auditctl -l` activity reappears; a previously observed `chmod` process event associated with Auditd session `33` reappears. This verifies those two saved-report sources respond in both directions to the dashboard picker, in addition to the already-validated telemetry source.
- **Remaining work:** Spot-check the other five saved-report sources, inspect any data-quality warnings, separate web request rows from server diagnostic lines, check SSH-to-Auditd session attribution, and export the final live Dashboard Studio source after all UI edits.

This is **functional time-picker validation**, not a claim that all saved searches are accurate or that SOC-003 has already been performed.

