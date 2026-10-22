# SOC Dashboard Studio — Three-tab source dashboard

The [Dashboard Studio JSON](soc-lab-dashboard-studio.json) is the maintained repository definition for the existing Splunk Cloud dashboard. The operator has shown the Grid layout working on all three tabs and has confirmed all eight reports in the `search` app. **The live dashboard now resolves all eight reports, and its exported source (including their time-range bindings) has been committed to this repository.** See the [report-save runbook](../docs/report-save-runbook.md) for the exact saved titles.

**Current status (October 21, 2026):** The three-tab dashboard and its source export are published; all eight report references render. The [SOC-003 investigation](../investigations/SOC-003/case-report.md) is complete and includes session-54 correlation evidence. The dated diagnostic steps farther down document earlier troubleshooting, not work still required to complete SOC-003. Exact saved-report SPL reconciliation and several time-filter edge cases remain open.

## Import into Splunk Cloud

1. Go to **Dashboards > Create New Dashboard**, choose **Dashboard Studio**, title **Splunk SOC Lab | Live Security Operations**, and choose a **grid layout**.
2. Save or open the dashboard in edit mode. Select the **Source code editor** icon.
3. Replace the dashboard definition with the **entire** contents of `soc-lab-dashboard-studio.json`, select **Apply**, and then **Save**. If the editor warns about unsupported layout or data sources, do not delete the original dashboard; capture the message for troubleshooting.
4. Open the **SOC Overview**, **Endpoint Activity**, and **Investigation Workspace** tabs and ensure the saved reports resolve.
5. In Dashboard Studio's data source picker, check that each saved report is attached from the **Search & Reporting** app (`search`) and the report's exact title. If you used a different report title, change the `ref` in the JSON to match.
6. **Time-range nuance:** The global time picker governs the three inline KPIs and the event trend. Saved report sources may inherit the report's own time range. In the Studio data source UI, choose **Any time range input on the page** and select the global input if available. Validate by changing from Last 24 hours to a smaller window. If saved report time pickers do not respond, use the UI instead of assuming the JSON's global settings apply to `ds.savedSearch`.
7. Disable auto-refresh, or keep any refresh cadence conservative until query costs and concurrent-search limits are understood. Non-scheduled `ds.savedSearch` data sources run an ad hoc search when loaded.
8. Save a new JSON export from Studio after making UI edits and commit the actual exported definition separately.

## Tabs

| Tab | Panels |
|---|---|
| SOC Overview | Total events, distinct observed hosts, minutes since newest indexed event; sourcetype time series; source inventory; SSH sessions; grouped audited file changes |
| Endpoint Activity | Sudo authorizations, selected audited process executions, web access, file-change operations |
| Investigation Workspace | SSH-to-Auditd session correlation; cross-source timeline; process and file evidence |

## Practical caveats

- Report title, owner, app, sharing and enabled status have been seen in an operator-provided Splunk Reports screenshot. The previously mismatched saved-report sources are now rendering in the operator's dashboard. Time-range filtering has been explicitly tested on telemetry, sudo and process reports; independent checks remain for the others.
- The `minutes since newest indexed event` metric measures data freshness, **not VM uptime, collection uptime, or SLA**.
- Source count, process execution count, and grouped file-operation count answer different questions.
- `LINUX-COR-001` produced session correlations in the lab, including the SOC-003 session-54 investigation. Broader mismatch/edge-case testing and exact saved-SPL reconciliation remain open; if a search is empty, check LOGIN/PID matching and time picker configuration rather than interpreting it as zero suspicious sessions.
- `LINUX-DET-002` only captures `sudo: ... COMMAND=` entries: Auditd `euid=0` alone does not establish human sudo invocation.
- `LINUX-DET-005` supports typical web access formats; records with `unparsed` fields need regex adjustment.
- These searches are intentionally report-based and **not active detection alerts**.

## Readiness checklist

- [x] Overview tab loads and last-event metric has plausible age in the captured test window.
- [x] All eight saved-report references resolve for the operator's `search` app dashboard.
- [ ] Sudo panel checked for actual `COMMAND=` content, or documented as no matching records.
- [x] Web panel shows parsed request rows and excludes the separately classified diagnostic line in the observed dashboard output.
- [x] SOC-003 session 54 SSH-to-Auditd correlation compared with raw LOGIN and authentication evidence.
- [ ] Changing time range affects the expected panels (verify saved report picker binding separately).
- [ ] All three tabs and relevant raw-data drilldowns reviewed.
- [x] Current Studio JSON exported from Splunk and stored in GitHub.
- [x] SOC-003 completed and published with an authorized Kali-to-Ubuntu login and labeled test file. See the [case report](../investigations/SOC-003/case-report.md).

## Grid adjacency fix — 2026-10-21

If Dashboard Studio displays **“Viz panel incorrectly configured”** and **“expected a viz or canvas edge directly below/above/left”**, the original Dashboard Studio JSON had blank coordinate gaps between grid panels (for example, a header ended at `y=130` but KPI panels started at `y=135`). Grid layout applies its own gutter spacing and requires each panel to touch another visualization or the canvas boundary in its positioning structure.

The checked-in JSON was repaired to use **continuous, edge-aligned rectangles** on all three tabs. The `gutterSize=8` layout option creates visible spacing *without coordinate gaps*. The positions were checked programmatically for overlaps/gaps and all eight saved-report references are preserved. This does not replace the need to re-import and confirm it in Splunk Cloud.

**Fix for a dashboard already in edit mode:** select **Source code**, replace the *entire* JSON with the latest [dashboard definition](soc-lab-dashboard-studio.json), then **Apply**. Do **not** alter the layout from Grid to Absolute to hide the warning. Check each tab for no red layout banners, and **Save** only after the editor accepts the revised definition. If other errors remain, capture the precise panel ID and message.

## Troubleshooting: source divergence and report resolution (2026-10-21)

### Evidence and current state

The production JSON on `main` has contiguous grid coordinates. A static geometry check found no gaps/overlaps and no dangling visualization data source IDs across three tabs. Yet operator screenshots from 2026-10-21 show old positions (`viz_total_events` begins at y=135, `viz_proc` y=470, `viz_session_correlation` y=125, and the file/process evidence split uses x=590/610). The production JSON instead uses y=130, y=445, y=115, and x=600/600. **Do not keep rewriting the committed coordinates** until the actual Splunk Source editor definition is compared. A GitHub commit does not automatically update an existing Splunk dashboard.

The separate saved-search errors show `Could not find object id=LINUX-DET-004...` and `LINUX-COR-001...`. Splunk documentation confirms `ds.savedSearch.options.ref` should be the **exact report name** and `options.app` should be the original app name (for Search & Reporting, `search`). The repository's `ref`/`app` schema is correct **provided** the actual report titles/app and viewing permissions match. We have not independently confirmed those saved-search objects within the operator's Splunk account.

### Diagnostic gate 1 — inspect the production source *before* changing it

1. Open the **existing** `Splunk SOC Lab | Live Security Operations` dashboard in Edit mode. Open **Source** (</> icon). Copy/export the entire current source to a private local backup before applying changes.
2. Find `"item": "viz_total_events"` in the **layout** `overview.structure` and read its `position.y`; expected committed value **130** (old **135**). Also inspect `viz_proc` (**445**, old **470**), `viz_session_correlation` (**115**, old **125**), and `viz_process_evidence` x (**600**, old **610**). Compare the exact fields; other `y` values elsewhere are normal.
3. **If old:** the dashboard is still using the previous definition. Replace the entire Source definition with the current [production JSON](soc-lab-dashboard-studio.json), click **Apply** and inspect each tab before **Save**. Save, reopen, recheck the values. Do not change Grid to Absolute just to suppress validation errors.
4. **If already current:** record the exact Source `layout` and any new validation errors; do not guess further coordinate changes. The static checker is not a replacement for Splunk's runtime validator.

### Diagnostic gate 2 — isolate saved search resolution

1. In **Search & Reporting > Reports**, inspect **LINUX-DET-003 - Monitored File Changes** and **LINUX-DET-004 - Audited Process Executions**. Record exact display name, App and Owner/Sharing; open them to confirm they are accessible. An empty result set is different from a missing saved-search object.
2. Within Dashboard Studio, use the **Data source overview** (cylinder icon) or select the relevant panel's data source. Choose **Saved search** and pick a report from the app/owner picker rather than typing a guessed title. Reopen/refresh the dashboard first if the report was created after opening the editor. Inspect Splunk-generated `ref` and `app`.
3. Compare the generated source with `ds_process` and `ds_correlation` in the production JSON. Correct only names/context that actually differ; fix access/permissions in Splunk if necessary. Do not convert every report to inline SPL on speculation.
4. If the root cause is still ambiguous, create a **separate temporary** Dashboard Studio dashboard named `SOC Lab | Dashboard Studio Diagnostic (Temporary)` and import [diagnostic-dashboard-studio.json](diagnostic-dashboard-studio.json). This grid has an intro, a left inline telemetry table, and a right saved file-change report table:
   - **Both work:** grid format and the known report reference work in this environment; recheck the production dashboard's applied source and the failing report names.
   - **Inline works, saved report is missing:** likely saved report name/app/owner/share/access issue; inspect the original report using the picker.
   - **Both show grid position errors:** capture the new exact errors and Splunk build/version for grid-layout compatibility review.
   - **Inline shows no events:** verify time range and live index separately; no events alone does not prove a grid/schema problem.
5. The diagnostic dashboard is intentionally disposable. **Do not** overwrite or delete the production dashboard, and do not mistake the diagnostic dashboard for production validation.

Official references:
- [Splunk Cloud: call saved searches](https://help.splunk.com/en/splunk-cloud-platform/create-dashboards-and-reports/dashboard-studio/9.3.2411/use-data-sources/use-reports-and-saved-searches-with-ds.savedsearch)
- [Splunk: Dashboard definition and multi-tab grid schema](https://help.splunk.com/en/splunk-cloud-platform/create-dashboards-and-reports/dashboard-studio/10.6/source-code-editor/what-is-a-dashboard-definition)

### Diagnostic gate 2 result — screenshot, 2026-10-21

The operator imported the temporary `diagnostic-dashboard-studio.json` into Splunk Cloud and shared a screenshot. **The test rendered with no red grid layout errors.** The inline `ds.search` panel returned counts for `syslog` (5401), `linux:audit` (3079), `linux_secure` (299), `soc:web:access` (5), plus a separate HEC test sourcetype (1). The right panel successfully displayed actual file-change rows using `ds.savedSearch` with `ref=LINUX-DET-003 - Monitored File Changes` and `app=search` (including commands shown as `bash`, `chmod`, `touch`).

**Implication:** the minimal grid structure, live inline SPL access, and at least one saved report reference work in this Splunk Cloud workspace. This does **not** prove the other seven saved searches resolve or that the production dashboard source is current. The input's *presence* does not prove time picker propagation.

**Historical next gate (since completed):** open the *production* three-tab dashboard (not the temporary Diagnostic dashboard) in Edit > Source, and compare the layout coordinates to `main`. Confirm `viz_total_events.position.y=130`, `viz_proc.position.y=445`, `viz_session_correlation.position.y=115` and `viz_process_evidence.position.x=600`. If old coordinates persist in Splunk, back up its source and apply the current checked-in production definition; then recheck. If coordinates already match, capture the full Source and precise validation errors before editing. Review failing individual saved reports by selecting them via the Studio report picker.

## Saved-search title mismatch resolved in source (2026-10-21)

An operator screenshot of **Search & Reporting > Reports** showed all eight Linux reports in the `search` app, owned by `sc_admin`, with `Private` sharing and `Enabled` status. The two functioning dashboard reports had matching titles. The remaining six failed because their actual saved titles were shorter and used an **em dash**, while the source JSON referenced different, longer titles with a regular hyphen.

The source dashboard was updated on `main` to reference these actual titles:

| Source key | Exact Splunk report title |
|---|---|
| `ds_telemetry` | `LINUX-MON-001 — Telemetry Health` |
| `ds_ssh` | `LINUX-DET-001 - SSH Authentication Sessions` |
| `ds_sudo` | `LINUX-DET-002 — Sudo Activity` |
| `ds_files` | `LINUX-DET-003 - Monitored File Changes` |
| `ds_process` | `LINUX-DET-004 — Process Execution` |
| `ds_web` | `LINUX-DET-005 — HTTP Requests` |
| `ds_correlation` | `LINUX-COR-001 — Session Correlation` |
| `ds_timeline` | `LINUX-INV-001 — Investigation Timeline` |

**Apply:** Back up the existing production dashboard Source locally, replace it with the newest [production JSON](soc-lab-dashboard-studio.json), click **Apply**, inspect all three tabs, then **Save**. Do not replace the dashboard with the *temporary diagnostic JSON*. The grid geometry and inline sources were not modified in the title-correction commit. Check that each panel stops showing `Could not find object id=...`, then test the reports' actual results and time picker. This title correction was subsequently applied in Splunk Cloud, and the live export now matches the checked-in report references.

**Note:** A report being `Private` is not itself evidence of an error in this owner's dashboard; two private reports were already rendering. If dashboards need to be shared with a different account later, change report/dashboard sharing according to the intended permissions.

### Completion criteria

Only consider the production dashboard ready after Grid errors disappear on every tab, **all eight saved report references** resolve, process/file/HTTP/correlation outputs are reviewed against actual SPL, time picker behavior is confirmed separately for inline and saved searches, and the saved dashboard survives reopen. The actual Studio source/export has been committed and SOC-003 has been completed. Remaining validation gaps are recorded in the [dashboard validation notes](../docs/dashboard-validation-2026-10-21.md).

## Live source export synchronized — 2026-10-21

The current [production JSON](soc-lab-dashboard-studio.json) is the **actual dashboard definition copied from Splunk Cloud** after the operator updated all eight saved-report time controls. Each saved report has a display name and uses Splunk-generated `queryParameters.earliest="$global_time.earliest$"`, `queryParameters.latest="$global_time.latest$"`, and `useSavedTimeRange=false`. The prior full-layout correction, visualization definitions and inline SPL were preserved. Splunk also exported a `ds.spl2` global query-parameter default.

The global time picker has been exercised against Telemetry Health, Sudo Activity and Process Execution with both short and full-day windows. The other five time-filtered report sources still need direct spot checks. HTTP request classification and SOC-003 session-54 SSH/Auditd attribution have since been demonstrated; wider tests and saved-SPL comparison remain open. See [validation findings](../docs/dashboard-validation-2026-10-21.md).
