# Splunk SOC dashboard

The Splunk Dashboard Studio section presents analyst-focused views of endpoint security telemetry and investigation activity.

## Dashboard layout

| Panel | Analyst question |
|---|---|
| Event volume over time | Is ingestion behaving as expected? |
| Events by source | Which log streams contribute most? |
| Active monitored hosts | Which endpoints are represented? |
| SSH authentication | Who is logging in, and from where? |
| Administrative activity | Which sudo and privileged events deserve review? |
| Process execution | Which audited processes are appearing? |
| File modification activity | What monitored paths have changed? |
| HTTP requests | What requests and response codes are recorded? |
| Detection findings | Which searches surfaced activity for review? |
| Investigation timeline | How do relevant events align chronologically? |

The historical synthetic index `soc_lab` and live telemetry index `soc-live` are treated as separate datasets.

<!-- SCREENSHOT: Dashboard overview -->
<!-- SCREENSHOT: Authentication and process analysis -->
<!-- EXPORT: Add Dashboard Studio JSON once built -->

For detection validation, proposed reports, panel SPL ideas, layout and stage gates, see the [detection-to-dashboard plan](../docs/detection-to-dashboard-plan.md). Dashboard Studio has not yet been configured or exported.

The prepared report bundle and detailed dashboard build order live in [the one-pass report runbook](../docs/report-save-runbook.md). Report-backed panels should use the saved report's exact name and app context after saving.
