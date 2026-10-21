# Portfolio demonstration — SOC-003

**Format:** 2–4 minute screen recording or annotated walkthrough using existing genuine Splunk evidence. No new event generation is necessary if the historical time range and screenshots are readable. Do not present still images as live monitoring.

## Storyboard

| Approx. time | Visual | Narration goal |
|---|---|---|
| 0:00–0:25 | Main README and architecture | Explain Ubuntu telemetry forwarded to Splunk Cloud, and the Kali workstation's role as authorized test source |
| 0:25–0:55 | SOC Overview screenshot | Explain source coverage and why ingestion freshness is not host health |
| 0:55–1:25 | SOC-003 SSH authentication screenshot | Identify source `172.16.110.128`, target, account and SSH PID `5613` |
| 1:25–1:55 | Auditd LOGIN screenshot | Show same-host PID and timestamp match assigning session `54` |
| 1:55–2:35 | Investigation Workspace evidence | Explain session-scoped processes and file operations; distinguish all-session context |
| 2:35–3:10 | Raw Auditd five-record screenshot | Show that five companion records represent one successful file-creation event |
| 3:10–3:35 | SOC-003 conclusion and limitations | Close as authorized activity, not a real compromise; note remaining validation work |

## Suggested narration

“I built this lab to practice the full SOC workflow, from collecting Linux logs through investigating activity and writing an evidence-supported conclusion. Ubuntu sends authentication, system, Auditd and HTTP events to Splunk Cloud through the Universal Forwarder. Kali is my authorized test workstation.

In SOC-003, I traced a successful SSH login from Kali to Ubuntu. The authentication log showed SSH process 5613. A matching Auditd LOGIN record on the same host and timestamp assigned session 54, allowing me to investigate process and file events belonging to that session.

The dashboard keeps session-specific evidence separate from the broader activity timeline. For a file creation, I inspected five raw Auditd records sharing one audit ID: they describe one operation, not five different file changes.

The result was authorized lab activity, not a compromise. I also document detection limitations, validation status, and the operational checks still outstanding.”

## Recording checklist

- Use the already-published original screenshots or actual Splunk dashboard.
- Confirm the selected time range contains October 21, 2026 evidence.
- Make text legible at typical 1080p viewing size; use restrained zooms and slow callouts.
- Never show passwords, tokens, deployment packages, or private Splunk URLs.
- Keep timestamps, field names, metrics and command output unchanged.
- Link the video to the SOC-003 report and repository README only after publication.
- Do not claim active alerting or production coverage.
