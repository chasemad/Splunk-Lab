# Linux SOC detection triage playbook

This playbook describes an analyst workflow for the controlled Splunk lab. It is not a claim that alerts are configured or continuously monitored.

## First-pass triage

1. Record the search/report name, exact UTC window, host, index, and raw event count.
2. Inspect the raw event and confirm the sourcetype, parsing, source host and timestamp.
3. Identify the question: authentication anomaly, privileged activity, audited execution, monitored file change, or HTTP request.
4. Search for related events by **host and defensible linkage fields**, not username or timestamp alone.
5. For SSH, preserve source IP, source port, SSH PID and account. Check Auditd LOGIN for a matching PID on the same host and in a plausible time window. Record the resulting `ses` only after confirmation.
6. For Auditd, group companion records by full `msg=audit(...)` event ID. Review SYSCALL success, PATH, CWD and PROCTITLE; distinguish `auid` from `euid`.
7. Compare with expected lab operations and known limitations. Do not infer maliciousness solely from a failed password, successful login, root-owned daemon, uncommon process, or HTTP 404.
8. Record disposition, evidence, alternative explanations and follow-up needs.

## Scenario decisions

| Scenario | Analyst questions | False-positive / ambiguity considerations |
|---|---|---|
| Repeated SSH failures | Are attempts within one transport connection or across several? Which user/IP/window? Was there a subsequent success? | Typing errors, expected administration, source NAT, incomplete auth formats |
| SSH success and endpoint actions | Does a matching Auditd LOGIN establish the session? Do the process/file events actually share it? | PID/session reuse, missing records, unrelated concurrent activity |
| Sudo authorization | Which account invoked sudo, what command was requested, and was it permitted? | Root service activity is not equivalent to sudo |
| Audited process execution | What executed, under which original and effective identities, and in which session? | Exec events do not prove command intent or downstream effect |
| Monitored file operation | What file, syscall, outcome and audit ID? Was it a create, write or attribute change? | Multiple companion records represent one event; watches cover only configured paths |
| HTTP requests | Is the record a valid parsed request? What status and URI? | Diagnostics are not requests; 404 is not intrinsically malicious |

## Disposition language

- **Authorized lab activity:** Directly attributable to planned, observed exercise.
- **Benign or expected:** Supported by context, without requiring attribution to a specific exercise.
- **Suspicious; investigate further:** Pattern warrants additional evidence but compromise is not established.
- **Inconclusive:** Collection gaps or ambiguous linkage prevent a defensible conclusion.

Avoid calling a search match a confirmed intrusion. Escalation depends on context and available evidence.

## Case note template

- Case ID / analyst / UTC window:
- Initial signal and SPL version:
- Data sources and ingestion caveats:
- Raw authentication evidence:
- SSH-to-Auditd correlation basis:
- Process and file event IDs:
- Competing explanations:
- Findings / disposition:
- Limitations and recommended follow-up:
- Evidence references:

See [SOC-003](../investigations/SOC-003/case-report.md) for a completed example and the [validation matrix](linux-report-validation-matrix.md) for test criteria.
