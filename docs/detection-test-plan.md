# Detection test plan — offline preparation

The tests below are specifications, **not executed test results**. Run them later only against authorized lab data or isolated, non-indexed synthetic fixtures. Use the [validation matrix](linux-report-validation-matrix.md) to record actual outcomes.

| Detection | Positive fixture | Negative / edge fixture | Expected interpretation |
|---|---|---|---|
| SSH session summary | Known accepted login with PID and source port | Unrelated SSH PID or port | Separate connection groups |
| Three-failure SSH candidate | Three failures followed by success in the same connection | One failure followed by success; failures after success | Only qualifying sequence matches; cross-connection attempts out of scope |
| Sudo activity | Explicit sudo authorization message | Root-owned SSH daemon LOGIN record | Only sudo authorization is shown |
| Process execution | Audited execve SYSCALL in known session | PATH companion record without an execution event | Avoid double-counting and false execution claims |
| Monitored file change | Successful watched file CREATE group | Unwatched path or unsuccessful syscall | Only matching successful monitored operations |
| HTTP requests | GET returning 200 or 404 | Server diagnostic line | Request-only view excludes diagnostic |
| SSH/Auditd correlation | Matching host/PID/time LOGIN record | Same PID on different host or distant time | Reject ambiguous attribution |
| Investigation timeline | Multiple correctly timestamped sources | Unrelated session events | Timeline retains context without implying same-session attribution |
| Telemetry health | Recent indexed events by source | No recent indexed events | Flag stale data; do not equate it with a proven host outage |

## Review gates

1. Confirm input fixture events and raw timestamps.
2. Run canonical SPL and capture result count and fields.
3. Compare actual behavior to expected behavior; document mismatches.
4. Inspect performance and earliest/latest handling.
5. Record known gaps (log formats, field extraction, PID reuse, session scope, event grouping).
6. Reconcile saved Splunk report SPL with the GitHub file before treating the repository as exact source of truth.
7. Configure scheduling only after tests and operational constraints are understood.

## Known tested examples

- SOC-003 Kali SSH PID 5613 → Auditd session 54 → watched file creation.
- Candidate three-failure query: known one-failure dataset produced no match; synthetic in-memory three-failure/one-success fixture produced a match.
- HTTP classification: four requests and one diagnostic line; corrected live panel showed requests only.

These examples do not establish complete detection coverage.
