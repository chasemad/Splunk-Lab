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
