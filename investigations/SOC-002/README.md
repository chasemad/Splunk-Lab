# SOC-002 — Linux endpoint activity investigation

## Investigation scope

This case study uses real telemetry generated within a controlled Kali Linux and Ubuntu Server lab. The investigation examines SSH authentication, Linux Auditd process and file records, administrative activity, and HTTP requests as related sources of endpoint evidence.

## Evidence sources

| Source | Investigation value |
|---|---|
| SSH authentication | Connection origin, account and authentication outcome |
| Auditd SYSCALL | Selected executed processes and audit context |
| Auditd PATH/CWD/PROCTITLE | Related file paths, working directories and command context |
| HTTP access log | Request origin, URI and response status |

## Investigation workflow

### Cross-source timeline

Use SPL to narrow authentication, Auditd and web activity into a readable event sequence while excluding routine background noise.

<!-- SCREENSHOT: Cross-source timeline -->

### Session attribution

Compare SSH activity with Auditd login identity (`auid`), audit session (`ses`), process identifiers and timestamps. Time proximity or matching usernames alone do not establish that an action originated in a particular SSH session.

<!-- SCREENSHOT: SSH and Auditd session evidence -->

### Audit event reconstruction

Group Auditd records using the event identifier inside `msg=audit(...)`. A SYSCALL, PATH, CWD and PROCTITLE record may describe the same operation.

<!-- SCREENSHOT: Grouped audit records -->

## Findings and disposition

<!-- COMPLETE AFTER STEP 42–44 AND INVESTIGATION: Insert actual event timeline, supporting evidence, analyst assessment, uncertainties, disposition, and recommended detection improvements. -->

## Detection opportunities

SSH authentication patterns, administrative activity, monitored file changes, selected process execution and unusual web requests are candidates for focused SPL searches.

See [Linux detection engineering](../../detections/linux/README.md).
