# Linux saved-report validation matrix

**Scope:** Eight reports saved in Splunk Cloud for `index="soc-live"`, host `soc-ubuntu-01`. This document distinguishes reported observations from tests still needed. Saved reports are **not** equivalent to scheduled alerts.

| ID | Intended question | Existing evidence | Next verification |
|---|---|---|---|
| LINUX-MON-001 | Which telemetry sources have indexed events recently? | Telemetry source rows and selected 15-minute/24-hour dashboard checks | Check time range, source counts and newest-event semantics across fresh and stale windows |
| LINUX-DET-001 | Which SSH connections had successes or failures? | Eight-event/seven-connection baseline; separate session-53 and session-54 evidence | Compare saved SPL; test password/public-key/invalid-user variants |
| LINUX-DET-002 | What sudo authorizations appear in auth.log? | Sudo panel populated in dashboard screenshots | Compare raw sudo messages to extracted user, command and result; ensure root processes are not mislabeled sudo |
| LINUX-DET-003 | Which watched file operations occurred? | Positive/negative pilot; SOC-003 three grouped IDs; one raw CREATE group reviewed | Verify audit-ID deduplication, write/attribute behavior and missing-event handling |
| LINUX-DET-004 | Which selected execve events occurred in audited sessions? | Session 54 process evidence includes identity and file-related commands | Compare saved SPL to raw SYSCALL/EXECVE records; verify uid/auid/session semantics |
| LINUX-DET-005 | Which HTTP requests and status codes were recorded? | Four request rows in the operator's October 21 report-editor screenshot; the diagnostic line is excluded | Screenshot-visible search text is synchronized with [GitHub SPL](../detections/linux/LINUX-DET-005-web-requests.spl); confirm Save/reopen persistence and test additional log formats |
| LINUX-COR-001 | Which SSH login corresponds to an Auditd LOGIN session? | SOC-003 PID 5613 and session 54 verified using raw evidence | Compare saved SPL; check unmatched logins, PID reuse, host/time constraints and join behavior |
| LINUX-INV-001 | What cross-source events occurred in the selected window? | Investigation timeline visible; all-session context distinguished from case-specific evidence | Verify ordering, source labels, global time range, and unrelated-session handling |

## Test record template

For each report record:

- **SPL version:** GitHub path/commit and exact Splunk saved-search text.
- **Time range:** UTC start and end; dashboard time picker value.
- **Fixture or evidence:** Source events and case identifier.
- **Positive expectation:** What should match and why.
- **Negative expectation:** What should not match and why.
- **Observed outcome:** Actual row count, key fields, and screenshot or exported evidence.
- **Limitations:** Data gaps, aggregation semantics, false positives, and search performance.
- **Disposition:** Pass, fail, or requires follow-up. Do not mark pass without observed output.

## Important validation rules

1. Distinguish raw Auditd companion record counts from unique audit event IDs.
2. Distinguish original login user ID (`auid`) from effective user ID (`euid`).
3. Treat source IP, host, SSH PID and matching Auditd LOGIN as a combination of evidence, not a single globally unique identifier.
4. Treat the global timeline as contextual; only session-filtered tables belong to the selected case.
5. Confirm whether a search is a report, a dashboard data source, or a scheduled alert.
6. Use controlled fixtures for negative and edge cases; do not infer coverage from a single successful example.
7. Do not put credentials, tokens, or private Splunk deployment packages into validation artifacts.

## Remaining access-dependent tasks

Exact saved-search SPL cannot be confirmed from GitHub alone. The LINUX-DET-005 source now matches the search text visible in the October 21 report-editor screenshot; the operator still needs to confirm it was saved by reopening that report. Export or copy the remaining seven searches when available. Runtime checks, Auditd persistence and forwarding behavior require access to the live environment.

Related: [saved report runbook](report-save-runbook.md), [dashboard validation](dashboard-validation-2026-10-21.md), [SOC-003 case](../investigations/SOC-003/case-report.md).
