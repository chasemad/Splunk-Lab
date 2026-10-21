# Linux detection backlog

**Status: All eight live reporting searches are saved in Splunk Cloud (operator-confirmed 2026-10-21). LINUX-DET-001 sessions and LINUX-DET-003 file-change positive/negative tests were previously reviewed; other report results and scheduled alerts still require verification.**

- [LINUX-DET-001 — SSH authentication baseline and candidate detection](LINUX-DET-001-ssh-auth.md) (session report saved; candidate three-failure alert not deployed).
- [LINUX-DET-002 — Sudo command activity](LINUX-DET-002-sudo-activity.spl) (new report SPL; validation pending).
- [LINUX-DET-003 — Monitored file changes (saved Report + tested pilot SPL)](LINUX-DET-003-file-change.md).
- [LINUX-DET-004 — Audited process executions](LINUX-DET-004-process-execution.spl) (new report SPL; validation pending).
- [LINUX-DET-005 — HTTP access activity](LINUX-DET-005-web-requests.spl) (live HTTP parsing corrected; GitHub SPL reconciliation pending).
- [LINUX-COR-001 — SSH-to-endpoint session correlation](LINUX-COR-001-ssh-endpoint-session.spl) (SOC-003 PID-to-LOGIN linkage demonstrated; saved-report SPL reconciliation pending).
- [LINUX-MON-001 — Telemetry source health](LINUX-MON-001-telemetry-health.spl).
- [One-pass report save checklist](../../docs/report-save-runbook.md).

Each detection must include source/field prerequisites, complete SPL, test cases, results, expected legitimate behavior, false positives, tuning and limitations. Do not infer a shared session solely from username or temporal proximity.
