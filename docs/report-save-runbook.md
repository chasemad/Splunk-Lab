# One-pass Splunk Cloud report creation pack

## Purpose and limitations

These SPL files are prepared for `index="soc-live"`, `host="soc-ubuntu-01"`. **Only LINUX-DET-003 is confirmed saved in Splunk Cloud**. LINUX-DET-001 SSH session summary was tested against real data, and a separate synthetic threshold test returned 3 failures/1 success. Other new report SPL is **draft, not run in Splunk yet**. Saving a report makes a search reusable; it does not mean that every intended field parsed correctly or that it is a validated alert.

A Splunk Cloud **free trial cannot use the management REST API**. Do these steps in **Search & Reporting** for each unsaved report: click file below, copy raw SPL, paste into Search, choose **Last 24 hours** (or a range containing known events), **Run**, examine fields, select **Save As > Report**, enter the exact report name, **Save**. Keep scheduling disabled; enable time-range picker when offered. If a report yields 0 events, save it only as an exploratory or baseline report and record why it is empty.

## Save in this order

| # | Splunk Report title | Canonical SPL | Status |
|---|---|---|---|
| 1 | `LINUX-MON-001 - Live Telemetry Health` | [SPL](../detections/linux/LINUX-MON-001-telemetry-health.spl) | To save |
| 2 | `LINUX-DET-001 - SSH Authentication Sessions` | [SPL](../detections/linux/LINUX-DET-001-ssh-session-summary.spl) | To save |
| 3 | `LINUX-DET-002 - Sudo Command Activity` | [SPL](../detections/linux/LINUX-DET-002-sudo-activity.spl) | To save |
| 4 | `LINUX-DET-003 - Monitored File Changes` | [SPL](../detections/linux/LINUX-DET-003-file-change.spl) | Already saved |
| 5 | `LINUX-DET-004 - Audited Process Executions` | [SPL](../detections/linux/LINUX-DET-004-process-execution.spl) | To save |
| 6 | `LINUX-DET-005 - HTTP Access Activity` | [SPL](../detections/linux/LINUX-DET-005-web-requests.spl) | To save |
| 7 | `LINUX-COR-001 - SSH Endpoint Session Correlation` | [SPL](../detections/linux/LINUX-COR-001-ssh-endpoint-session.spl) | To save |
| 8 | `LINUX-INV-001 - Live Investigation Timeline` | [SPL](../investigations/SOC-003/live-investigation-timeline.spl) | To save |

### Report notes

**LINUX-MON-001 - Live Telemetry Health:** Operational visibility: event counts and freshest timestamp per indexed log source. Historical counts do not imply a currently healthy forwarder.

**LINUX-DET-001 - SSH Authentication Sessions:** Summarize accepted and failed SSH authentication by connection; baseline already tested with 8 events and 7 connections.

**LINUX-DET-002 - Sudo Command Activity:** Actual sudo command authorization records from auth.log; does not claim every root process represents a human sudo action.

**LINUX-DET-003 - Monitored File Changes:** Audited successful write/attribute changes grouped by audit event ID. Saved in Splunk Cloud; no need to create twice.

**LINUX-DET-004 - Audited Process Executions:** Auditd process execution events with original/effective IDs and session context; execve success does not prove command intent or subsequent filesystem success.

**LINUX-DET-005 - HTTP Access Activity:** Web request investigation view; parser handles common log style and some key=value variants, and visibly flags unparsed formats.

**LINUX-COR-001 - SSH Endpoint Session Correlation:** Candidate correlation linking SSH pid from auth.log with Auditd LOGIN pid and audit session; tests needed for real output and join limits.

**LINUX-INV-001 - Live Investigation Timeline:** Three-source analyst event timeline; not a direct proof that timestamp-adjacent events share an origin.

## Quick confirmation before opening Dashboard Studio

- [ ] LINUX-MON-001 has at least one known source row; time values are plausible and the time range covers ingestion.
- [ ] LINUX-DET-001 session summary contains 7 SSH connection groups from the previously captured 8 auth events when using the same time window.
- [x] LINUX-DET-003 file-change report saved in Splunk (user confirmed).
- [ ] LINUX-DET-002 sudo parse checked against at least one auth.log command record (or recorded as no data).
- [ ] LINUX-DET-004 audit session/process/euid fields inspected against raw examples.
- [ ] LINUX-DET-005 method/URI/status extraction checked against actual access.log entries. If values display `unparsed`, adapt regex before relying on charts.
- [ ] LINUX-COR-001 returns plausible matched SSH and Auditd sessions (if none, troubleshoot subsearch match + timestamps; don't force a fake match).
- [ ] LINUX-INV-001 timeline sorts across auth, process, watched file operations and HTTP data.
- [ ] Saved report names, app context, sharing permissions and time ranges recorded for dashboard setup.

## Dashboard plan — next phase

Open **Dashboards > Create New Dashboard > Dashboard Studio**, use Grid layout, and set a time picker. Add panels in stages:

1. Telemetry health table and last-seen KPIs via `LINUX-MON-001`. A timestamp of the most recently *indexed event* does not guarantee a service is currently healthy.
2. SSH connection summary via `LINUX-DET-001`; successes/failures chart can use a separate inline timechart rather than aggregating pre-aggregated connection rows.
3. Sudo activity via `LINUX-DET-002`, audit processes via `LINUX-DET-004`, and grouped file operations via `LINUX-DET-003`.
4. HTTP table via `LINUX-DET-005` after confirming parsing.
5. SSH-to-Auditd correlation via `LINUX-COR-001`; the analyst investigation timeline via `LINUX-INV-001`.

In Dashboard Studio, use **Saved search / report** as the data source where possible (Splunk `ds.savedSearch`); select the correct app context (`search` for Search & Reporting), owner and exact name. Save dashboard source JSON, review queries, and capture actual screenshots. Non-scheduled reports may execute dynamically and are subject to concurrent search limits. Keep expensive investigation searches off auto-refresh if performance is poor.

## SOC-003 — controlled live Kali investigation

Do a controlled, documented sequence **only against your owned Ubuntu lab VM**:

1. Take an initial monitoring snapshot; note current UTC time and that the Auditd rules and Splunk forwarder are running. Verify existing SSH connectivity rather than assuming any URL or port.
2. From the Kali VM, establish an approved SSH session to Ubuntu as `socadmin`, noting time and source address.
3. In that session, run benign identity checks (`whoami`, `id`); create a labeled file under `/opt/soc-lab-test/`, change its mode, then append a line; exit. Do not target system-sensitive files.
4. If the existing lab web server is running, perform a simple GET to its already configured local URL. Do not guess the port.
5. In Splunk, use `LINUX-DET-001` to find the authenticated connection, `LINUX-COR-001` for the derived audit session, `LINUX-DET-004` for audited commands, and `LINUX-DET-003` for the file activity. Inspect raw records using matching audit IDs for disposition.
6. Write `SOC-003` with exact UTC timeline, identities, PID/session linkage, screenshots, benign context, detection matches/misses, and uncertainty. Correctly classify the authorized experiment, not as a real adversary intrusion.

**Avoid storing passwords, Splunk deployment credentials, tokens, non-redacted user information, or generic offensive test payloads in the repository.**
