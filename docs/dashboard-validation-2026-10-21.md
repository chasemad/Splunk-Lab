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

## Remaining validation and deliverables

- [x] Three tabs render without the previously reported grid or unresolved-report errors.
- [x] Actual Dashboard Studio JSON export stored in GitHub.
- [x] Global saved-report time bindings present for all eight sources in JSON.
- [x] Telemetry, sudo and process time-range behavior tested against short and longer windows.
- [ ] Spot-check SSH, file changes, HTTP, session correlation and investigation timeline time-range behavior.
- [ ] Validate HTTP parsing and distinguish diagnostic log lines from requests.
- [ ] Independently validate correlation attribution and relevant process/file events.
- [ ] Save/reopen and confirm the latest state persists after edits.
- [ ] Compare the actual saved Splunk reports with the canonical SPL and document detection positive/negative tests.
- [ ] Capture and commit sanitized evidence screenshots.
- [ ] Conduct distinct SOC-003 investigation on owned lab VMs and publish case findings.

This is a controlled laboratory demonstration, not a production intrusion or claim of enterprise SOC experience.
