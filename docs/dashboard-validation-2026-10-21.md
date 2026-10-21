# Dashboard Studio validation — 2026-10-21

**Project:** Splunk SOC Lab | Live Security Operations  
**Scope:** live Linux telemetry from `soc-ubuntu-01`, Splunk `index="soc-live"`  
**Status:** Dashboard renders across all three tabs; report references resolve; global saved-report time binding configured for all eight sources in the exported JSON. Some report behavior and detection findings remain to be tested.

## Baseline: three-tab runtime screenshots

Operator screenshots showed functioning SOC Overview, Endpoint Activity and Investigation Workspace tabs, with no reported missing-object or Grid configuration banners.

| Check | Screenshot finding | Interpretation |
|---|---|---|
| SOC Overview | 8,834 events; 1 host; newest event age approximately 1 minute | Point-in-time baseline, not continuous uptime |
| Telemetry inventory | syslog 5,421; linux:audit 3,104; linux_secure 303; soc:web:access 5; manual HEC test 1 | Sum = **8,834**, consistent with KPI in the same screenshot |
| Endpoint Activity | Sudo commands, process executions, file operations and HTTP access rows | Saved searches resolve and show meaningful fields |
| Investigation Workspace | Candidate SSH/Auditd session correlation and timeline plus file/process tables | Tables render, but attribution needs confirmation |

The raw screenshots still need to be added to the repository in sanitized form.

## Time-picker diagnosis and fix

### Initial failure

Selecting **Last 15 minutes** reduced the inline total-events KPI from its earlier 24-hour value of 8,834 to 36 and updated the event-volume chart, but the saved-report tables continued to show older activity.

The Dashboard Studio data-source editor showed the cause: the saved source was configured to **Inherit from saved search**, using the report's own Last 24 hours range.

### Native UI correction

For each of the eight saved-report sources, the operator selected **Any time range input on the page** and assigned a display name. The input is titled **Investigation time range** and uses token `global_time`.

The uploaded **actual Splunk Dashboard Studio Source export** was synchronized to the repository at [soc-lab-dashboard-studio.json](../dashboards/soc-lab-dashboard-studio.json). Each `ds.savedSearch` source now has a display name and the following Splunk-generated configuration:

```json
{
  "queryParameters": {
    "earliest": "$global_time.earliest$",
    "latest": "$global_time.latest$"
  },
  "useSavedTimeRange": false
}
```

All eight report `ref` names and app `search` remain unchanged. The three-tab layout, visualization definitions and existing inline SPL queries were preserved. Splunk also exported a `ds.spl2` default with the same global earliest/latest parameters.

### Observed tests (not a guarantee of every report)

- **Telemetry Health:** Under Last 15 minutes, showed Auditd 27, syslog 21 and authentication 4 = **52** events, with first/last timestamps inside the short selected period. Earlier Last 24 hours inventory showed 8,834 events; screenshots were taken at different times.
- **Sudo Activity:** Returned *No search results* for Last 15 minutes, and earlier `auditctl -l` activity returned at Last 24 hours.
- **Process Execution:** Returned *No search results* for Last 15 minutes, and an earlier successful `chmod` event in Auditd session 33 returned at Last 24 hours.
- **Yellow warning icon:** On the short-window process panel, hover text was **No search results returned**—not a syntax error.
- **Other five sources:** All contain the correct token-binding configuration in the export; their bidirectional runtime filtering has not yet been independently checked.

## Outstanding data-quality and attribution work

### DV-002 — HTTP diagnostic line

The HTTP table includes valid GET request rows with statuses 200 and 404, but also one `unparsed` line resembling `code 404, message File not found`. The canonical `LINUX-DET-005` search intentionally displays unmatched lines as `unparsed`. Separate server diagnostic lines from actual HTTP requests before interpreting request counts or status rates. Test a revised search against the observed raw events before changing the saved Splunk report.

### DV-003 — SSH/Auditd correlation scope

The candidate correlation reported SSH PID **5124**, source IP **172.16.110.128**, Auditd session **33**, **73** audited execution `SYSCALL` records and **3** selected monitored-file `SYSCALL` records. These counts do **not** necessarily correspond to 73 distinct manually typed commands or represent malicious behavior.

A separate unfiltered file-evidence panel displayed Auditd session **6**, not session 33. **Do not attribute those file rows to the correlated SSH session.** Verify login PID-to-Auditd LOGIN assignment, timestamps, session boundaries, raw event IDs and associated file operations before relying on the correlation as case evidence. Improve workspace filtering/labeling so general endpoint activity isn't mistaken for selected-session evidence.

### DV-004 — Panel readability

Long `_raw` fields and multivalue process lists crowd the tables, particularly in Investigation Workspace. Consider concise default columns, clearly labeled timezone and an optional raw-event drilldown before final recruiter screenshots.

### DV-005 — Saved search contents and permissions

The Splunk Reports page showed eight enabled Linux reports, app `search`, owner `sc_admin`, and `Private` sharing. The telemetry SPL was inspected in Studio and matched the canonical repository search visually. The remaining saved SPL searches have **not** been exported and systematically compared byte for byte to the repository. Private reports may not be accessible to other Splunk accounts.

## HTTP cleanup and session-filter pilot — 2026-10-21

The operator provided four new Splunk Cloud screenshots after following the HTTP report and session-filter candidate instructions:

- **HTTP classification:** Within a 24-hour search over `/opt/soc-web-logs/access.log`, five source events were classified as **four HTTP requests** and **one diagnostic_or_unparsed** record.
- **HTTP dashboard:** Endpoint Activity's **Web request activity** panel displayed parsed GET requests, including `/missing-page` with **404**, `/third-test.txt` with **200**, and `/` with **200**. The previously visible separate unparsed diagnostic row was not present in the shown portion of the panel. The saved report's precise SPL/export still requires confirmation before synchronizing the canonical GitHub SPL.
- **Session input:** The revised Investigation Workspace was visible in Edit mode with an **Audit session (case filter)** number input set to **33**, and the selected correlation table returned SSH PID `5124`, Auditd session `33`, `audited_execs=73`, `monitored_file_ops=3`. This confirms the selected correlation renders for session 33; the counts and attribution still require raw-log examination.
- **Session 6 comparison:** Changing the Audit session input to **6** displayed **No search results returned** in the selected SSH-to-Auditd correlation table. The independently labeled **Recent activity timeline — all sessions** continued to display background records. Empty correlation is an acceptable outcome if no qualified login/PID match exists; it is not evidence of missing session-6 file activity.
- **Pending:** Capture the *bottom evidence tables* at session **33** and again at **6**, to show that their returned `audit_session`/`sessions` fields match the input. Verify no cross-session leakage and that the all-session timeline deliberately remains broader. Then save, reopen, and export Splunk's actual revised JSON to synchronize GitHub. The operator later exported the working dashboard and its complete JSON was committed to the repository on 2026-10-21. The candidate is no longer pending synchronization.

These are operator-supplied screenshots of a controlled lab, not confirmed enterprise incident evidence. Publication screenshots still need browser/account chrome cropped and review for sensitive details.

## Selected-session evidence tables validated — 2026-10-21

Three operator screenshots validated the revised *Investigation Workspace* with a case-specific **Audit session** input.

- **Session 33, selected file evidence:** monitored operations involving `/opt/soc-lab-test/det003-validation.txt`; every visible `sessions` field equals **33**. The displayed `processes` include `bash` and `chmod`; the records show successful operations.
- **Session 33, selected process evidence:** `chmod`, `cat`, `dircolors`, and `dirname` shown with `audit_session=33`, including process identifiers and user identity fields. Additional records may be paginated.
- **Session 6, selected file evidence:** previously observed operations on `/opt/soc-lab-test/third-test.txt` and `/opt/soc-lab-test/evidence.txt`, each with `sessions=6`. **No session 33 rows** are visible in these filtered tables.
- **Session 6, selected process evidence:** `auditctl`, `grep`, `sudo`, and `ls` are visible with `audit_session=6`. Different effective user IDs are present; do not equate an effective root context automatically with unauthorized activity.
- **Session 33 overview:** the selected correlation table still shows source `172.16.110.128`, SSH PID **5124**, Auditd session **33**, with candidate counts of **73** audited `SYSCALL` executions and **3** monitored file-operation `SYSCALL` records. These still require separate source-level attribution/count review.
- **Timeline distinction:** `Recent activity timeline — all sessions` remains visible for broad chronological context and explicitly states it is *not* filtered by the selected Audit session. Do not attribute every background row to the selected connection.
- **Presentation:** the third screenshot shows a full Investigation Workspace in View mode (green **Edit** button), including the session input, correlation, all-session timeline, and both case-filtered evidence tables. The post-change dashboard Source export was received and committed. The green **Edit** button in the View mode screenshot indicates a rendered saved view; a separate close/reopen persistence test has not been explicitly recorded.

**Result:** The requested 33-versus-6 filter test passed on visible file/process rows. The post-save Dashboard Studio Source export has now been compared with the previous canonical JSON and committed to GitHub. Existing eight saved-search references and Overview/Endpoint layouts were preserved. Genuine sanitized images still need to be included in the repository to replace evidence placeholders.

## Remaining validation and deliverables

- [x] Three tabs render without the previously reported grid or unresolved-report errors.
- [x] Actual Dashboard Studio JSON export stored in GitHub.
- [x] Global saved-report time bindings present for all eight sources in JSON.
- [x] Telemetry, sudo and process time-range behavior tested against short and longer windows.
- [ ] Spot-check SSH, file changes, HTTP, session correlation and investigation timeline time-range behavior.
- [x] Validate HTTP parsing classification (four request lines and one diagnostic among five records) and visually confirm clean dashboard HTTP rows.
- [ ] Export the exact updated saved HTTP report SPL from Splunk and reconcile the canonical GitHub SPL.
- [x] Visually confirm selected-session filter changes file/process evidence between session 33 and 6.
- [ ] Independently validate the underlying SSH PID/Auditd LOGIN attribution and count semantics.
- [x] Export the actual post-session-filter Dashboard Studio Source JSON and commit it as canonical.
- [ ] Explicitly test closing/reopening the dashboard to confirm persistence.
- [ ] Compare the actual saved Splunk reports with the canonical SPL and document detection positive/negative tests.
- [ ] Capture and commit sanitized evidence screenshots.
- [ ] Conduct distinct SOC-003 investigation on owned lab VMs and publish case findings.

This is a controlled laboratory demonstration, not a production intrusion or claim of enterprise SOC experience.

## Final session-filtered dashboard export synchronized — 2026-10-21

The user uploaded the full 636-line Dashboard Studio Source definition after applying the two session-filtering changes and testing session 33 versus session 6. GitHub's [`dashboards/soc-lab-dashboard-studio.json`](../dashboards/soc-lab-dashboard-studio.json) was updated from that exported definition (commit `76dedc7`). Validation of the submitted definition found **no out-of-bounds panels, overlaps, uncovered Grid cells, missing visualization source IDs, or missing input IDs**. It preserves all eight saved-report sources and their global time inputs while adding a numeric case session selector (`investigation_session`, default `33`) plus three session-scoped inline searches (`ds_case_correlation`, `ds_case_files`, `ds_case_process`).

The value `33` is a reproducible example from the recorded lab session, **not a permanent unique identifier** across host reboots or all future investigations. Analysts must confirm the host, time window, SSH PID and Auditd LOGIN before treating a session number as attributable to a connection. The all-session timeline intentionally remains wider than the case-filtered evidence.
