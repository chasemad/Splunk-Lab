# SOC-002 — Live Linux Endpoint Investigation

## At a glance

**Investigation question:** Can I connect a remote login to the processes and file creation that followed, using real Linux logs?

**What I did:** Traced an authorized Kali SSH login to Auditd session **27**, identified `whoami`, `id` and `touch` execution, and examined the audit records confirming a file was created in the test directory.

**What it demonstrates:** Security monitoring, cross-source event correlation, audit-log interpretation, timeline reconstruction, and evidence-based case closure.

**Outcome:** The events were consistent with the controlled exercise and the case was closed as **authorized lab activity**. Unlike SOC-001, this case uses **real logs collected from the lab VMs**. For a screenshot-backed walkthrough of the same investigation method, see [SOC-003](../SOC-003/case-report.md).

**Investigation type:** Controlled lab-generated security investigation  
**Status:** Closed — authorized test activity  
**Investigation date:** 2026-10-21 (UTC)  
**Environment:** Splunk Cloud / Ubuntu Server / Kali Linux  
**Data source:** Live endpoint logs forwarded to `index="soc-live"`

> **Portfolio context:** This report documents a controlled exercise on personally managed virtual machines. The activity is not evidence of a real-world attack or enterprise SOC case.

## Executive summary

An SSH connection from the Kali laboratory VM (`172.16.110.128`) authenticated successfully to the monitored Ubuntu server (`soc-ubuntu-01`, `172.16.110.130`) using the `socadmin` account. The server assigned Linux audit session `27` to the connection. Several seconds later, the session recorded execution of `whoami`, `id`, and `touch`; correlated Auditd file-operation records show successful creation of `/opt/soc-lab-test/soc002-test.txt`.

The observed actions were intentionally generated for telemetry and investigation validation. The investigation was therefore closed as **authorized lab activity**. If comparable events appeared in a production environment without an established legitimate explanation, an analyst would need to validate the account owner, source, administrative context, and purpose of the file operation before disposition.

## Objective and scope

The objective was to use multiple real log sources—not synthetic CSV data—to establish a defensible sequence from remote authentication to endpoint process activity and file creation. The search was conducted manually as a lab investigation; **it was not triggered by a deployed correlation alert**.

| Item | Value |
| --- | --- |
| Investigated host | `soc-ubuntu-01` |
| Investigated source | `172.16.110.128` (Kali VM, lab NAT network) |
| Destination host | `172.16.110.130` (Ubuntu VM) |
| Account | `socadmin` |
| Original audit user ID | `auid=1000` |
| SSH daemon PID | `4937` |
| Linux audit login session | `ses=27` |
| Monitored file | `/opt/soc-lab-test/soc002-test.txt` |
| Splunk index | `soc-live` |
| Reviewed sources | `/var/log/auth.log`, `/var/log/audit/audit.log` |

## Evidence-based timeline

Times shown are UTC on **2026-10-21**, as displayed in the corresponding ingested log records.

| Time (UTC) | Event | Supporting evidence |
| --- | --- | --- |
| **11:11:32.811** | SSH password authentication accepted for `socadmin` from `172.16.110.128` | `/var/log/auth.log`: `sshd[4937]` with `Accepted password` |
| **11:11:32.811** | Audit session `27` assigned to `socadmin` | Auditd `LOGIN`: `pid=4937`, `auid=1000`, `ses=27` |
| **11:11:39.459** | `whoami` process execution | Auditd execution record: `comm="whoami"`, `auid=1000`, `ses=27`, `euid=1000` |
| **11:11:39.460** | `id` process execution | Auditd execution record: `comm="id"`, `auid=1000`, `ses=27`, `euid=1000` |
| **11:11:39.461** | `touch` process execution | Auditd event `1792581099.461:893` with `SYSCALL` and `EXECVE`; target filename present in arguments |
| **11:11:39.461** | Test file created | Auditd event `1792581099.461:894`: `SYSCALL` with `success=yes`, `syscall=56` (`openat` on ARM64), `exit=3`, and `PATH` with `nametype=CREATE` for `/opt/soc-lab-test/soc002-test.txt` |

## Technical findings

### Finding 1 — Remote authentication mapped to an audit login session

The SSH log and Auditd `LOGIN` record share `pid=4937` and timestamp `11:11:32.811`. The authentication log identifies `172.16.110.128` as the SSH source and `socadmin` as the account. The Auditd login record assigns `auid=1000` and `ses=27`, establishing the context for subsequent audited processes. The shared PID is the crucial link; timestamp proximity alone would be insufficient.

### Finding 2 — Process executions belong to session 27

Auditd records for `whoami`, `id`, and `touch` show `auid=1000`, `ses=27`, and `euid=1000`. These executions ran with the normal effective privileges of `socadmin`, rather than root privileges. Other session processes may have elevated effective IDs as part of operating-system login setup and should not be classified as deliberate `sudo` activity without examining their context.

### Finding 3 — File creation has its own Auditd operation

The `touch` executable's `execve` event has audit identifier `1792581099.461:893`. The file operation has a distinct audit identifier, `1792581099.461:894`. The second event comprises related `SYSCALL`, `PATH`, `CWD`, and `PROCTITLE` records.

The `SYSCALL` result reports `success=yes`, `comm="touch"`, `ses=27`, and `syscall=56`, the ARM64 `openat` system call. Its `exit=3` indicates a returned file descriptor on success—not a process exit status of 3. The `PATH` record identifies the target file with `nametype=CREATE`, supporting the conclusion that the test file was created.

### Finding 4 — Raw record counts are not operation counts

Auditd can emit multiple records for a single system call. The grouping key `msg=audit(<timestamp>:<serial>)` lets an analyst assemble related `SYSCALL`, `PATH`, `CWD`, `EXECVE`, and `PROCTITLE` records. Counting every raw record as a distinct command or file change would overstate activity.

## Analyst assessment and disposition

**Disposition: Authorized laboratory test / no unauthorized compromise identified in the reviewed activity.**

The account login, commands, and file creation were deliberately initiated to demonstrate SIEM ingestion and correlation. The commands `whoami`, `id`, and `touch` are not independently malicious; the file was created in a designated lab test directory. No evidence reviewed here establishes an unauthorized actor, malicious objective, or production impact.

For an otherwise unexplained event sequence on a real server, investigation would be appropriate. Before closing such a case, an analyst should corroborate identity and authorization, inspect the account's expected access, examine command purpose and file path sensitivity, and review related endpoint/network activity. An unusual time or a command run with elevated privileges can raise priority, but neither proves compromise.

**MITRE ATT&CK consideration:** The `whoami` and `id` commands are compatible with general account or system discovery behaviors, but this controlled activity is not being attributed to an adversary or assigned a confirmed malicious technique.

## Investigation limitations

- This is an intentionally generated lab sequence, not an alert-driven incident.
- The evidence links the SSH process to audit session 27 and audited execution within that session; it does not independently authenticate the human operating Kali.
- Auditd `execve` captures process execution, not every shell built-in, complete terminal interaction, or business intent.
- The findings concern selected records and one monitored test file; they do not establish that all activity on the host was reviewed.
- File creation was established by audit records. Independent filesystem inspection or content verification is outside this documented evidence set.
- Log timestamps and Splunk's display timezone should be checked when recreating timeline evidence.
- Audit rules were initially created with `auditctl`; persistence across reboot remains separate infrastructure work.

## Evidence and screenshot placeholders

Add **real, sanitized Splunk screenshots** before publishing. Suggested relative paths:

- `../../evidence/SOC-002/01-ssh-authentication.png` — SSH `Accepted password` record for `sshd[4937]`.
- `../../evidence/SOC-002/02-login-session-27.png` — Auditd `LOGIN` event mapping PID 4937 to session 27.
- `../../evidence/SOC-002/03-session-processes.png` — `whoami`, `id`, `touch` records showing `ses=27` and `auid=1000`.
- `../../evidence/SOC-002/04-file-creation-audit.png` — Event `1792581099.461:894` and `success=yes` / `nametype=CREATE`.
- `../../evidence/SOC-002/05-investigation-timeline.png` — Saved cross-source investigation search, after a final screenshot is captured.

Do not publish cloud credentials, HEC tokens, private keys, or unsanitized sensitive logs.

## Reproducible Splunk searches

**SSH authentication:**

```spl
index="soc-live" source="/var/log/auth.log"
"Accepted password" "172.16.110.128"
| table _time _raw
| sort 0 _time
```

**Audit login mapping:**

```spl
index="soc-live" source="/var/log/audit/audit.log"
"type=LOGIN" "ses=27"
| table _time _raw
```

**Session process executions:**

```spl
index="soc-live" source="/var/log/audit/audit.log"
"type=SYSCALL" "key=\"soc_lab_exec\"" "ses=27"
| rex field=_raw "comm=\"(?<process>[^\"]+)\""
| rex field=_raw "euid=(?<effective_uid>\d+)"
| table _time process effective_uid _raw
| sort 0 _time
```

**File creation audit operation:**

```spl
index="soc-live" source="/var/log/audit/audit.log"
"1792581099.461:894"
| rex field=_raw "type=(?<record_type>[A-Z_]+)"
| table _time record_type _raw
| sort 0 _time
```

## Follow-up engineering work

1. Develop narrow Linux detections for SSH authentication, unexpected privilege use, selected file changes, and process patterns, grounding logic in available telemetry.
2. Test each detection with controlled positive and negative events; document expected administrative false positives.
3. Create an investigation-focused dashboard with authentication, Auditd, file activity, and telemetry health views.
4. Make audit rules persistent after reviewing their scope and volume; verify Splunk forwarding across restarts and rotation.
5. Include this case and supporting screenshots in the portfolio repository with the limitations clearly stated.

---

**Detection follow-up:** [LINUX-DET-003 — Monitored file changes](../../detections/linux/LINUX-DET-003-file-change.md).
