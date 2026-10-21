# SOC-003 — Kali-to-Ubuntu SSH Session Investigation

| Case attribute | Value |
|---|---|
| Case ID | SOC-003 |
| Investigation date | 2026-10-21 |
| Classification | Controlled security-monitoring exercise |
| Analyst disposition | **Authorized laboratory activity; no confirmed compromise** |
| Status | **Case narrative complete**; final screenshot publication and remaining optional follow-up validation pending |
| Source workstation | Kali Linux (`luxieum`), `172.16.110.128` |
| Monitored endpoint | Ubuntu (`soc-ubuntu-01`), `172.16.110.130` |
| Login identity | `socadmin` (Auditd original login ID `auid=1000`) |
| SIEM / index | Splunk Cloud / `soc-live` |
| Primary evidence sources | `/var/log/auth.log`, `/var/log/audit/audit.log` |
| Dashboard | [Splunk SOC Lab — Dashboard Studio](../../dashboards/soc-lab-dashboard-studio.json) |

> This case documents activity that the lab operator deliberately performed on personally controlled virtual machines. It is a demonstration of analyst methodology, not a claim of an external attack or enterprise compromise. All timestamps below are **UTC on October 21, 2026**, as shown in the operator's screenshots.

## Executive summary

A controlled SSH connection was established from Kali (`172.16.110.128`) to a monitored Ubuntu server (`172.16.110.130`) as `socadmin`. Authentication was accepted at **14:32:20.566 UTC**. The SSH server reported PID **5613**, and the Ubuntu Auditd `LOGIN` record reported that same PID at the same timestamp, assigning **audit session 54**. This provided a host, PID, timestamp and session-based linkage from authentication to subsequent Linux audit telemetry.

The investigator selected session **54** in a three-tab Splunk Dashboard Studio workspace and observed audited process activity (`whoami`, `id`, `chmod`, `stat`) plus monitored file activity against `/opt/soc-lab-test/soc003-kali-validation.txt`. A focused raw Auditd search returned **five companion records for one file-creation audit ID**, proving successful file creation under the selected session. A separate local/self-SSH test with two failures followed by one success was observed earlier in **session 53** and is not treated as part of the Kali-originated connection.

**Disposition:** The evidence is consistent with known, authorized lab operations. No unauthorized access, privilege abuse, or malicious intent is established by the observed records. The exercise validates the ability to reconstruct a traceable sequence across authentication, Auditd LOGIN, selected process execution, and audited file creation.

## Scope and collection

- **In scope:** operator-managed Kali and Ubuntu VMs on a VMware Fusion NAT lab network; Splunk Cloud searches/dashboards; the controlled files in `/opt/soc-lab-test/`.
- **Out of scope:** other hosts or networks; external accounts; threat actor attribution; claims about unknown activity outside the observed window.
- **Preflight:** operator screenshots showed SSH and Auditd active, Auditd `lost=0`, `soc_lab_file_change` and `soc_lab_exec` rules present, the Splunk Universal Forwarder running, and the test directory owned by `socadmin`. Kali-to-Ubuntu connectivity was confirmed. Splunk source inventory showed current `linux:audit`, `linux_secure`, and `syslog` data.
- **Scope of detection output:** the dashboard's `audited_execs` metric and `monitored_file_ops` metric are aggregate search results. They are not automatically a count of manually entered commands, nor proof of malicious behavior.

## Evidence-based timeline

| Time (UTC) | Observation | Source / key linkage |
|---|---|---|
| 14:25:13 | First failed password attempt for `socadmin` | Ubuntu-originated SSH test, `172.16.110.130:43388`, `sshd[5463]` |
| 14:25:18 | Second failed password attempt | Same source, port and SSH PID |
| 14:25:26 | Successful authentication; Auditd assigns session **53** | Same local/self-SSH test; not the Kali session |
| 14:32:20.566 | **Kali-originated password authentication accepted** for `socadmin` | `/var/log/auth.log`, source `172.16.110.128:54272`, `sshd[5613]` |
| 14:32:20.566 | **Auditd LOGIN assigns session 54** | `/var/log/audit/audit.log`, `pid=5613`, `auid=1000`, `ses=54`, audit ID `1792593140.566:1414` |
| ~14:32:28 | Selected-session dashboard lists identity and file-related processes | Session 54; visible processes include `whoami`, `id`, `chmod`, `stat` |
| 14:32:28.187 | **Successful file creation** at `/opt/soc-lab-test/soc003-kali-validation.txt` | Raw audit ID `1792593148.187:1492` (5 companion records) |
| 14:32:28.188 | Additional grouped `chmod` and `bash` file-operation records | Session 54, audit IDs `1792593148.188:1494` and `1792593148.188:1495`; not independently inspected as raw event groups |

Times shown as approximate or to the second in visualization panels must not be mistaken for microsecond-exact command timing. The detailed Auditd record timestamps are the stronger source for individual audited actions.

## Finding 1 — SSH authentication bound to Auditd LOGIN

The Kali-source authentication event from `/var/log/auth.log` reported:

```text
sshd[5613]: Accepted password for socadmin from 172.16.110.128 port 54272 ssh2
```

The matching `type=LOGIN` record in `/var/log/audit/audit.log` reported `pid=5613`, `auid=1000`, `ses=54` at **14:32:20.566 UTC** and the Auditd message ID `1792593140.566:1414`. The host, PID and timestamp agreement support assigning subsequent Auditd session-54 evidence to this login within the observed window.

The `uid=0` on a privileged `LOGIN` record describes its process context; it does **not** mean the authenticated interactive user ran the subsequent commands as root.

### Authentication context: separate session 53

The SSH summary also contained **2 failures and 1 success**, `failure_then_success=Yes`, for PID **5463**, source `172.16.110.130`, source port **43388**. The matching Auditd LOGIN assigned **session 53**, and the subsequent file is named `soc003-validation.txt`.

The source address is the Ubuntu host itself. These events are useful as a controlled failed-then-successful authentication example, **not** evidence of attacks from Kali or part of the separate PID-5613/session-54 sequence. The Kali-source connection had **one observed success and no observed preceding failures in that connection**.

## Finding 2 — Audited process activity under session 54

The Investigation Workspace was set to **Audit session 54**, and its process evidence table showed session-54 rows for:

| Visible command | Interpretation |
|---|---|
| `whoami` | Identity verification |
| `id` | User/group identity verification |
| `chmod` | Changed permissions of controlled test file |
| `stat` | Inspected the controlled file's metadata |

The visible records used `audit_user=1000` and `effective_uid=1000`, aligning with `socadmin`. Some other execution activity was present in the dashboard. The candidate correlation showed **77 audited execution SYSCALL records**, which include selected system and shell-related activity; they are **not** 77 independently verified typed commands.

Observed commands are consistent with the planned exercise and do not establish privilege escalation.

## Finding 3 — Raw Auditd evidence of controlled file creation

**Raw audit event ID:** `1792593148.187:1492`  
**Timestamp:** 14:32:28.187 UTC  
**File:** `/opt/soc-lab-test/soc003-kali-validation.txt`

A direct Splunk search for the complete audit ID returned **five Auditd records**:

| Type | Raw evidence observation |
|---|---|
| `PATH` | `item=1`, filename `/opt/soc-lab-test/soc003-kali-validation.txt`, `nametype=CREATE`, owner/group 1000 |
| `PATH` | `item=0`, parent `/opt/soc-lab-test/`, `nametype=PARENT` |
| `CWD` | `cwd="/home/socadmin"` |
| `SYSCALL` | `syscall=56` (`openat` on this ARM64 environment), `success=yes`, `ses=54`, `auid=1000`, `uid=1000`, `euid=1000`, `comm="bash"`, `exe="/usr/bin/bash"`, `key="soc_lab_file_change"`, `pid=5670`, `ppid=5671` |
| `PROCTITLE` | `proctitle="-bash"` |

**Interpretation:** These are **five components of a single successful audited file-creation event**, rather than five separate file writes. Shell output redirection reasonably explains why Auditd attributes the file creation to `bash` rather than a standalone `printf` process. The `PATH` `nametype=CREATE` and companion successful `SYSCALL` make the creation conclusion stronger than the grouped dashboard row alone.

The selected-session dashboard separately showed two more successful grouped operations involving this file:

- `1792593148.188:1494` — `chmod` (likely permission update; raw companions not independently opened)
- `1792593148.188:1495` — `bash` (consistent with a second shell write; raw companions not independently opened)

The dashboard's `monitored_file_ops=3` is consistent with these **three audit event IDs**, but the raw record set was independently reviewed only for **:1492**.

## Dashboard observations and detection coverage

| Capability | Observed result | Notes |
|---|---|---|
| SSH reporting (LINUX-DET-001) | Kali session displayed with 0 failures / 1 success; independent local session with 2 failures / 1 success | Aggregated by connection attributes |
| Session correlation | SSH PID 5613 → Auditd LOGIN session 54 | Validated against separate raw auth and LOGIN searches |
| Process evidence | Session 54 commands visible | Selected-session evidence differs from general machine activity |
| File evidence | Session 54 file rows visible; raw CREATE group checked | Three file-operation IDs, one fully raw-reviewed |
| Investigation timeline | Broader context includes other sessions | **Not** filtered to session 54; chronology alone is not attribution |
| Monitoring/ingestion | Active auth, Auditd and syslog events | Point-in-time evidence, not an uptime guarantee |
| HTTP requests | Not part of this case sequence | Previously validated elsewhere in the lab |

## Analyst assessment and disposition

**Finding:** Consistent, authorized SSH login and expected file activity in a controlled training environment.

**Disposition:** **Benign / authorized lab activity — close SOC-003 investigation.** No containment, credential reset, or external escalation is supported by the known lab context and the observed results.

**Reasoning:**

- The operator initiated the Kali login, performed the test operations on an owned endpoint, and supplied corroborating screenshots.
- Authentication and Auditd LOGIN share host, PID and timestamp; the Auditd session ID ties the selected process/file rows to that login.
- The raw Auditd `PATH` + `SYSCALL` companions show successful file creation in the approved test directory under non-root original/effective user ID 1000.
- The separate failed-login sequence originated from Ubuntu itself and is not misrepresented as Kali activity.
- No evidence presented establishes an unapproved user, out-of-scope file, persistence mechanism, unauthorized elevation, or malicious execution.

### Limitations / remaining quality checks

1. The two later file-operation IDs (`:1494`, `:1495`) have **grouped dashboard evidence**, but their complete raw companion records were **not individually reviewed**. This does not weaken the directly observed `:1492` creation conclusion.
2. The correlation's **77** audited executions is an aggregate of selected audit events, not 77 human-issued commands. Likewise, counts of raw Auditd records should not be confused with counts of separate operations.
3. A numeric Auditd session ID such as **54** is not globally unique over all hosts/reboots. The session linkage here depends on host, event time, SSH PID and LOGIN evidence.
4. Source data is visible in supplied operator screenshots; raw files were not acquired as independent forensic images, and report/dashboard access remains private in Splunk Cloud.
5. These are laboratory observations rather than production alert firing, active incident response, or proof of an external intrusion.
6. The case's screenshot set still needs **sanitization and publication into GitHub**. Avoid screenshots that expose unrelated account/session information.

## Reproducible searches

### Successful Kali authentication

```spl
index="soc-live" host="soc-ubuntu-01" source="/var/log/auth.log"
"Accepted password" "172.16.110.128"
| rex field=_raw "sshd\[(?<ssh_pid>\d+)\]"
| table _time ssh_pid _raw
| sort 0 -_time
```

### Matching Auditd LOGIN

```spl
index="soc-live" host="soc-ubuntu-01" source="/var/log/audit/audit.log"
"type=LOGIN"
| rex field=_raw "(?:^|\s)pid=(?<ssh_pid>\d+)"
| rex field=_raw "(?:^|\s)ses=(?<audit_session>\d+)"
| search ssh_pid="5613"
| table _time ssh_pid audit_session _raw
| sort 0 -_time
```

### Complete raw file-creation group

```spl
index="soc-live" host="soc-ubuntu-01" source="/var/log/audit/audit.log"
"1792593148.187:1492"
| rex field=_raw "type=(?<record_type>[A-Z_]+)"
| table _time record_type _raw
| sort 0 _time
```

## Evidence catalog — images to publish

Capture original screenshots with relevant time picker, source, fields, PID, Auditd session and event IDs visible. Crop unrelated browser tabs and account controls. These images were viewed during the investigation but **are not yet committed to GitHub**.

| Planned sanitized image | Purpose |
|---|---|
| `01-kali-ssh-authentication.png` | Successful auth from 172.16.110.128, SSH PID 5613 |
| `02-auditd-login-session54.png` | PID 5613 → Auditd session 54 |
| `03-ssh-session-summary.png` | Compare separate local failed-then-successful session 53 to Kali session 54 |
| `04-session54-correlation.png` | Dashboard session 54, PID/source and candidate correlation |
| `05-session54-process-and-file-evidence.png` | Selected session 54 file and process results |
| `06-raw-file-creation-audit-id.png` | Five companion Auditd records proving CREATE and successful openat |

Files should be stored in an appropriate dedicated `investigations/SOC-003/screenshots/` directory after review. Do not fabricate evidence imagery.

## Skills demonstrated

Splunk SPL, security log ingestion review, Linux SSH authentication analysis, multi-record Auditd interpretation, PID/session-based cross-source correlation, investigation scoping, process/file provenance analysis, evidence-driven triage, and reasoned incident closure.
