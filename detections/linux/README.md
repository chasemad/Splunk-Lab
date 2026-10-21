# Linux detection backlog

**Status: a file-change pilot search is documented; the first live Linux file-change Report is saved (operator-confirmed); remaining reports and scheduled alerts require further testing.**

- [LINUX-DET-001 — SSH authentication baseline and candidate detection](LINUX-DET-001-ssh-auth.md) (8 observed auth events; tuning and saved report pending).
- LINUX-DET-002 — Unexpected sudo or administrative activity.
- [LINUX-DET-003 — Monitored file changes (saved Report + tested pilot SPL)](LINUX-DET-003-file-change.md).
- LINUX-DET-004 — Selected process executions.
- LINUX-DET-005 — HTTP access anomalies.
- LINUX-COR-001 — SSH followed by related endpoint activity, subject to verified session attribution.

Each detection must include source/field prerequisites, complete SPL, test cases, results, expected legitimate behavior, false positives, tuning and limitations. Do not infer a shared session solely from username or temporal proximity.
