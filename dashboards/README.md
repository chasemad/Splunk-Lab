# Splunk Dashboard Studio — Live Security Operations

The [exported three-tab dashboard](soc-lab-dashboard-studio.json) provides a practical investigation workspace for Linux telemetry collected in Splunk Cloud from the controlled Ubuntu lab.

| Tab | Analyst purpose | Key views |
|---|---|---|
| SOC Overview | Check indexed telemetry and review authentication patterns | Event volume, source inventory, host/event counts, SSH sessions, grouped file operations |
| Endpoint Activity | Examine administrative and endpoint behavior | Sudo, audited process executions, monitored file operations, HTTP requests |
| Investigation Workspace | Correlate one authenticated session to subsequent endpoint activity | SSH-to-Auditd correlation, session-scoped process/file evidence, all-session contextual timeline |

## Investigation workflow

1. Select a time window containing the known activity.
2. Identify the relevant SSH connection and its source IP and process ID.
3. Confirm the matching Auditd LOGIN record before using the Audit session filter.
4. Enter the assigned session number to inspect case-specific process and file evidence.
5. Review the broader timeline separately; events in that timeline are not necessarily attributable to the selected login.
6. Inspect raw Auditd record groups when an individual file operation requires confirmation.

**Important:** Session numbers are not globally unique; use host, timestamp and matching LOGIN evidence for attribution. Dashboard counts reflect the underlying SPL aggregation and do not necessarily count human-entered commands.

## Configuration and validation

- [Working Dashboard Studio JSON](soc-lab-dashboard-studio.json) — exported from Splunk Cloud on October 21, 2026.
- [Setup and troubleshooting](dashboard-studio-setup.md) — layout and data-source configuration.
- [Validation record](../docs/dashboard-validation-2026-10-21.md) — time picker, saved-report references, HTTP parsing and session-filter tests.
- [Saved report runbook](../docs/report-save-runbook.md) — eight exact report titles and canonical SPL references.
- [SOC-003 case report](../investigations/SOC-003/case-report.md) — complete session 54 investigation with evidence.

The eight saved-report sources use the global time input rather than their individually saved search windows. The HTTP request panel excludes a diagnostic line that is retained in the raw index. Additional report-by-report tests and exact saved-SPL reconciliation are still outstanding.

The [diagnostic dashboard JSON](diagnostic-dashboard-studio.json) is retained as troubleshooting history and is **not** the working dashboard export.

## Screenshot gallery

These are curated captures from the real Splunk environment. Cropping and captions improve readability without replacing Splunk data.

### 01 — SOC Overview

![SOC Overview: telemetry sources, SSH sessions and file operations](screenshots/01-soc-operations-overview.png)

*Live source inventory, SSH activity, and grouped file operations.*

### 02 — Endpoint Activity

![Endpoint Activity: sudo, process, file and HTTP panels](screenshots/02-endpoint-telemetry.png)

*Endpoint investigation panels backed by saved reports.*

### 03 — Investigation Workspace

![Session-scoped process and file evidence](screenshots/03-session-scoped-evidence.png)

*Selected-session evidence is separate from the broader all-session timeline.*

### 04 — HTTP classification test

![HTTP request and diagnostic record classification](screenshots/04-http-request-classification.png)

*Four request records and one diagnostic/unparsed record were distinguished during parser validation.*

### 05 — Corrected HTTP request view

![Parsed HTTP requests and response codes](screenshots/05-clean-http-request-view.png)

*The dashboard shows recognized HTTP requests without presenting the server diagnostic as an additional request.*

## Remaining operational checks

Exact saved-report SPL comparison, complete time-range behavior tests, Auditd rule persistence, forwarder restart/log rotation, and alert scheduling remain separate tasks. See the [project roadmap](../docs/roadmap.md).
