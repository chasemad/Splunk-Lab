# Linux detection backlog

**Status: LINUX-DET-003 report saved (operator-confirmed), LINUX-DET-001 session summary tested, remaining report searches packaged for batch manual save; scheduled alerts remain unconfigured.**

- [LINUX-DET-001 — SSH authentication baseline and candidate detection](LINUX-DET-001-ssh-auth.md) (8 observed auth events; tuning and saved report pending).
- [LINUX-DET-002 — Sudo command activity](LINUX-DET-002-sudo-activity.spl) (new report SPL; validation pending).
- [LINUX-DET-003 — Monitored file changes (saved Report + tested pilot SPL)](LINUX-DET-003-file-change.md).
- [LINUX-DET-004 — Audited process executions](LINUX-DET-004-process-execution.spl) (new report SPL; validation pending).
- [LINUX-DET-005 — HTTP access activity](LINUX-DET-005-web-requests.spl) (parser validation pending).
- [LINUX-COR-001 — SSH-to-endpoint session correlation](LINUX-COR-001-ssh-endpoint-session.spl) (candidate session linkage; validation pending).
- [LINUX-MON-001 — Telemetry source health](LINUX-MON-001-telemetry-health.spl).
- [One-pass report save checklist](../../docs/report-save-runbook.md).

Each detection must include source/field prerequisites, complete SPL, test cases, results, expected legitimate behavior, false positives, tuning and limitations. Do not infer a shared session solely from username or temporal proximity.
