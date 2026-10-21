# SOC-002 — Live Linux endpoint investigation

**Status: in progress. Real, controlled lab-generated telemetry; no incident conclusion yet.**

## Verified evidence
- Kali → Ubuntu SSH authentication captured in `/var/log/auth.log`.
- Auditd process execution and file-change records ingested.
- Python HTTP test server received GET requests with 200 and 404 responses.
- Earlier process summary: 167 matching syscall records, 58 distinct process names.

## Next investigation steps
**Step 42:** construct a cleaner chronological view of SSH authentication, selected Auditd SYSCALL records and web requests; filter irrelevant CRON noise.

**Step 43:** examine SSH activity from lab Kali and compare with Auditd `auid`, `ses`, `pid`, and `ppid` where available. Do not assume matching timestamps establish session attribution.

**Step 44:** group Auditd record types using `msg=audit(...)` event identifiers so SYSCALL, PATH, CWD and PROCTITLE records for one operation are not counted separately.

## Reporting outline
Executive summary; scope; detection trigger; hosts/accounts; sources; UTC timeline; observed facts vs hypotheses; technical findings; ATT&CK mappings only where justified; uncertainty; disposition; recommendations; detection improvements.

**[PLACEHOLDER: Step 42 cross-source timeline screenshot]**

**[PLACEHOLDER: SSH-to-Auditd session attribution evidence]**

**[PLACEHOLDER: grouped Auditd records]**
