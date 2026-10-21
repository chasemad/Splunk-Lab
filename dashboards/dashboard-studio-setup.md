# SOC Dashboard Studio — Three-tab source dashboard

The [Dashboard Studio JSON](soc-lab-dashboard-studio.json) is a **prepared definition**, not a claimed deployment to Splunk Cloud. It references all eight report names from the [report-save runbook](../docs/report-save-runbook.md), assuming reports live in the `search` app and are visible to the dashboard owner.

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

- **Report names and permissions are not directly visible to ChatGPT inside Splunk.** User has confirmed eight saved reports, but this JSON has not been executed in their Cloud workspace.
- The `minutes since newest indexed event` metric measures data freshness, **not VM uptime, collection uptime, or SLA**.
- Source count, process execution count, and grouped file-operation count answer different questions.
- `LINUX-COR-001` correlation was only prepared in the repository and still requires real output validation; if empty, audit LOGIN/PID matching and time picker configuration rather than interpreting it as zero suspicious sessions.
- `LINUX-DET-002` only captures `sudo: ... COMMAND=` entries: Auditd `euid=0` alone does not establish human sudo invocation.
- `LINUX-DET-005` supports typical web access formats; records with `unparsed` fields need regex adjustment.
- These searches are intentionally report-based and **not active detection alerts**.

## Readiness checklist

- [ ] Overview tab loads and last-event metric has plausible age.
- [ ] Each report resolves with correct `search` app access.
- [ ] Sudo panel checked for actual `COMMAND=` content, or documented as no matching records.
- [ ] Web panel parses client IP, HTTP method, path and status.
- [ ] SSH-to-Auditd correlation returns a known session and is compared with raw LOGIN and authentication logs.
- [ ] Changing time range affects the expected panels (verify saved report picker binding separately).
- [ ] All three tabs and relevant raw-data drilldowns reviewed.
- [ ] Current Studio JSON exported and stored in GitHub.
- [ ] SOC-003 performed with one authorized Kali-to-Ubuntu login and labeled test file. See [SOC-003 investigation guide](../investigations/SOC-003/README.md).

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

### Completion criteria

Only consider the production dashboard ready after Grid errors disappear on every tab, **all eight saved report references** resolve, process/file/HTTP/correlation outputs are reviewed against actual SPL, time picker behavior is confirmed separately for inline and saved searches, and the saved dashboard survives reopen. Capture actual Studio source/export for GitHub after validation. SOC-003 remains pending until this gate passes.
