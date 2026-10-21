# Linux detection backlog

**Status: planned; none validated yet.**

- LINUX-DET-001 — SSH authentication anomalies.
- LINUX-DET-002 — Unexpected sudo or administrative activity.
- LINUX-DET-003 — Monitored file modifications.
- LINUX-DET-004 — Selected process executions.
- LINUX-DET-005 — HTTP access anomalies.
- LINUX-COR-001 — SSH followed by related endpoint activity, subject to verified session attribution.

Each detection must include source/field prerequisites, complete SPL, test cases, results, expected legitimate behavior, false positives, tuning and limitations. Do not infer a shared session solely from username or temporal proximity.
