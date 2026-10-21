# SOC-003 — Controlled Kali-to-Ubuntu Investigation

**Status:** Planned, not executed yet.

This case will trace one authorized Kali SSH session across Splunk authentication logs, Auditd session/process events, monitored file changes, and optional existing lab web access. See the [one-pass report pack](../../docs/report-save-runbook.md).

## Evidence to collect

- [ ] A single reference start time in UTC and host/network context.
- [ ] SSH auth log source account, IP, SSH PID and time.
- [ ] Auditd LOGIN with the same SSH PID and assigned `ses`.
- [ ] Process exec events from that `ses`.
- [ ] Monitored test file `PATH`, `SYSCALL`, and associated audit event ID.
- [ ] If applicable, controlled HTTP request/response from existing lab web server.
- [ ] Analyst decision and false-positive discussion.

Use [live investigation timeline SPL](live-investigation-timeline.spl). Do not represent this planned activity as an observed incident until after executing and reviewing the lab test.
