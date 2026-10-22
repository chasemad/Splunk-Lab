# From live investigations to detections, reports, and Dashboard Studio

This document preserves the original build plan for the live `soc-live` dataset. The lab now has eight saved Linux reports, a published three-tab dashboard and the completed SOC-003 case. **The proposed report names below are historical planning labels, not the exact titles currently saved in Splunk.** For the actual titles and query files, use the [saved-report inventory](report-save-runbook.md). Historical `soc_lab` detections (DET-001–004, COR-001) remain separate.

## Standard process for every Linux rule

1. **Specify behavior and scope** — Identify what would justify triage, which Ubuntu logs contain the evidence, and how an analyst would distinguish normal administration.
2. **Write exploratory SPL** — Extract `auid`, `ses`, `comm`, IPs, HTTP status and Auditd event IDs only where the relevant records contain them. Use bounded windows.
3. **Validate** — Generate safe, controlled positives on the two owned VMs, plus expected negatives. Capture the actual matching events, number of distinct underlying operations, and any misses.
4. **Tune** — Record authorized baseline behavior, high-volume processes, and false-positive scenarios. Prefer precise combinations of user, path, process, source, and context over blanket exclusions.
5. **Save as Report** — In Splunk Search & Reporting, click Save As > Report; use the name listed below. Record description, intended time range, full SPL and screenshot. Reports are not automatically alerts.
6. **Decide whether to alert** — After repeatable tests and accounting for ingestion lag, permissions, search schedule, overlapping windows, duplicates and noise, create an alert only if supported by the Splunk plan.
7. **Connect dashboard panels** — Display independently reproducible counts, last-seen timestamps, and drilldowns. Never label a matching search result a confirmed incident without analyst review.

## Original build order and proposed saved-report names

| Order | ID | Proposed Splunk report | Inputs | Validation target |
|---|---|---|---|---|
| 1 | LINUX-DET-003 | LINUX-DET-003 - Monitored File Changes | Auditd SYSCALL + PATH | One event per audited file operation, successful create/modify, normal reads excluded |
| 2 | LINUX-DET-001 | LINUX-DET-001 - SSH Authentication Activity | auth.log | Separate successful/failed logins; grouping by source IP and user; no invented failure counts |
| 3 | LINUX-DET-002 | LINUX-DET-002 - Administrative Activity | auth.log + Auditd | Expected sudo/login-init versus actual unexpected elevated actions |
| 4 | LINUX-DET-004 | LINUX-DET-004 - Selected Process Executions | Auditd SYSCALL + EXECVE | Normal tool usage, argument context, parent process if present |
| 5 | LINUX-DET-005 | LINUX-DET-005 - Web Request Review | HTTP access log | Legitimate 200, missing-page 404; exact IP, URI, status extraction |
| 6 | LINUX-COR-001 | LINUX-COR-001 - SSH to Endpoint Activity | auth.log + Auditd LOGIN/SYSCALL/PATH | Correlate SSH PID with Auditd LOGIN session, then with process/file events |

### Correlation rule prerequisite

First derive authenticated SSH events from `auth.log` including `sshd[pid]`. Join to a matching Auditd `LOGIN` record **using that PID and host**, and then aggregate relevant SYSCALL/PATH records by the resulting `ses`. Preserve timestamps and avoid accidental PID reuse over long periods. Username/time proximity alone is not session attribution.

## Original stage gates (completion varies by test)

**Gate A: pilot rule validated** — Re-run LINUX-DET-003, capture positive create + modify, negative read-only, record false positives and exact SPL in GitHub.

**Gate B: core detections validated** — Finish SSH and administration reports, including normal session initialization and expected sudo behavior. Add process execution and HTTP review.

**Gate C: correlation validated** — Replicate SOC-002 session 27 association with a query not hardcoded to session 27; demonstrate a separate new login and its session assignment.

**Gate D: saved reports ready for visualization** — All selected SPL searches saved with matching GitHub exports, tested time ranges and documented query costs.

**Gate E: dashboard** — Use Splunk Dashboard Studio. Build a useful analyst experience before adding decorative panels.

## Dashboard Studio panels and suggested source logic

| Panel | What to show | Query idea / caveat |
|---|---|---|
| Ingestion health | Last event by source, count in selected window | `index="soc-live" \| stats latest(_time) as last_seen count by host source sourcetype` |
| Event volume | Events per interval | `index="soc-live" \| timechart span=15m count by sourcetype` (syslog may dominate) |
| Hosts monitored | Hosts represented and last-seen time | `stats latest(_time) as last_seen by host`; a count of seen hosts is **not** uptime |
| SSH activity | Accepted and failed logins by hour, account and IP | Extract from `/var/log/auth.log`; distinguish invalid users and different failure formats |
| Admin activity | Reviewable sudo/PAM events, authenticated account | Distinguish `sudo` logs from system login scripts |
| Process activity | Top audited commands, distinct sessions | Deduplicate by Auditd event ID if combining record types |
| File changes | Successful monitored operations and watched paths | Use validated LINUX-DET-003 grouped output |
| Web activity | Requests by source, URI and status | Validate actual `soc:web:access` parsing; Python server diagnostic 404 lines are not independent requests |
| Detection review | Hits from validated reports, by detection ID | Do not invent alert state; the search may be report-only |
| Investigation timeline | Filterable notable events for SOC-002 style cases | Preserve source, user, session and timestamps; show raw drilldown |

### Suggested layout

- **Top row:** overall ingestion count, last event, distinct hosts (small KPI tiles).
- **Row 2:** event volume over time plus volume by source.
- **Row 3:** SSH authentication and administrative activity.
- **Row 4:** process executions and successful file operations.
- **Row 5:** HTTP status codes and reviewed detection matches.
- **Bottom:** investigation/event timeline with source-aware drilldown.

In Splunk, open **Dashboards > Create New Dashboard > Dashboard Studio**. Select a grid layout, choose the dashboard time range, add a data source for each panel, test the SPL in Search & Reporting first, then pick a visualization. Prefer restrained colors, consistent time labels, and readable tables. Save and export the Studio definition JSON after confirming queries and permissions. Capture genuine screenshots.

## Infrastructure / operational preflight

Before trusting the dashboard or alerts, check the Ubuntu root disk/LVM capacity, Auditd rule persistence, Universal Forwarder behavior after log rotation/restart, service account read permissions, time synchronization, index retention and noisy syslog ingestion. Remove stale HEC tokens and unnecessary credential-package copies. Restrict all test traffic to owned VMs.

## Repository deliverables per detection

- A versioned `.spl` file or full SPL block.
- A Markdown case sheet with objective, source prerequisites, test cases, observed counts, noise/false positives, changes made during tuning and limits.
- Scrubbed screenshots and source/time-range notes.
- A corresponding named Splunk Report (when actually created), and Dashboard Studio panel after stage gates pass.
