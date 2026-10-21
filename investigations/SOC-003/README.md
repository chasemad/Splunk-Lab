# SOC-003 — Controlled Kali-to-Ubuntu Investigation

**State:** SOC-003 core SSH/Auditd session attribution and session-filtered file/process evidence captured. Raw companion-record verification and the final case report remain pending.
**System:** Kali test workstation `172.16.110.128` → Ubuntu `172.16.110.130` (`soc-ubuntu-01`) → Splunk Cloud `index="soc-live"`.
**Purpose:** Demonstrate authorized SSH access, audited identity commands, controlled file changes, cross-source correlation, and an evidence-backed analyst disposition.

## Stage 0 — preflight

Only perform this on your personally managed training VMs. Confirm Ubuntu, Kali, Auditd, SSH, and the Universal Forwarder are running; confirm the Splunk dashboard has been imported and at least one source is updating. The persistent Auditd rule and forwarder behavior across reboots are separate checks: consult [Auditd rules](../../telemetry/auditd-rules.md) and [forwarder configuration](../../telemetry/splunk-forwarder-config.md). Keep screenshots free of credentials and unrelated browser content.

## Stage 1 — generate one safe, traceable session

From the **Kali terminal**:

```bash
date -u "+KALI_START_UTC=%Y-%m-%dT%H:%M:%SZ"
ssh socadmin@172.16.110.130
```

Use your normally authorized SSH credentials. **Inside the Ubuntu shell reached via SSH**:

```bash
date -u "+UBUNTU_SESSION_UTC=%Y-%m-%dT%H:%M:%SZ"
whoami
id
printf 'SOC-003 authorized training activity\n' > /opt/soc-lab-test/soc003-validation.txt
chmod 640 /opt/soc-lab-test/soc003-validation.txt
printf 'Second expected file update\n' >> /opt/soc-lab-test/soc003-validation.txt
stat /opt/soc-lab-test/soc003-validation.txt
exit
```

Do not target sensitive operating system files or other systems. The shell, not `printf`, may appear as the writing process in Auditd due to output redirection. Create/change/append should yield relevant audited operations if the temporary file watch remains loaded. `stat` is an inspection command; do not count it as another modification.

If the existing local training web server is already running, a normal HTTP GET to its previously configured local address/port can be included; confirm the configured port instead of assuming one. Record its status code.

## Evidence checkpoint — 2026-10-21 (operator screenshots)

Screenshots from **Search & Reporting** document a controlled test on 2026-10-21:

1. **Separate local/self-SSH example:** Two failed password logins followed by one accepted password for `socadmin` at 14:25:13, 14:25:18 and 14:25:26 UTC. The `/var/log/auth.log` events shared `sshd[5463]`, source `172.16.110.130` and source port `43388`. A matching Auditd `type=LOGIN` for PID `5463` assigned **session 53** at 14:25:26 UTC. Because the source equals Ubuntu's host IP, this is **not evidence of Kali-originated authentication**, even though the terminal application was accessed through the Kali desktop.
2. **Kali identity:** The Kali shell separately displayed hostname `luxieum`, source address `172.16.110.128`, current user `root`; the Ubuntu target was `172.16.110.130`.
3. **SOC-003 Kali-to-Ubuntu login:** `/var/log/auth.log` showed `Accepted password for socadmin from 172.16.110.128 port 54272`, recorded as `sshd[5613]` at **14:32:20.566 UTC**.
4. **Cross-source match:** `/var/log/audit/audit.log` yielded one `type=LOGIN` row with `pid=5613`, `auid=1000`, **`ses=54`**, at **14:32:20.566 UTC**, and audit message ID **`1792593140.566:1414`**. This is the observed PID/time bridge from the Kali-authenticated connection to an Auditd login session. The `uid=0` on the LOGIN record belongs to the privileged daemon context and is not, by itself, proof that the interactive user ran commands as root.
5. **Next case-evidence gate:** Set Investigation Workspace's **Audit session (case filter)** to **54** and choose a time window covering 14:32 UTC. Check candidate correlation, filtered audited process evidence and filtered monitored-file evidence. Look for `soc003-kali-validation.txt` **only if the file commands were actually executed during this separate Kali-originated session**. If a panel is empty, inspect raw records instead of assuming the event occurred.

**Do not combine** session 53's two failures and accepted login with session 54's Kali login as though they were a single remote session. Both are valid controlled lab observations with different originating hosts. Correlation here establishes a plausible session linkage; final analyst disposition still requires checking the process/file audit IDs and observed command sequence.

## SOC-003 session 54 evidence checkpoint

Operator screenshots captured from the working Dashboard Studio on 2026-10-21:

- SSH authentication: Accepted password for `socadmin` from Kali `172.16.110.128` to Ubuntu `172.16.110.130` at 14:32:20.566 UTC, `sshd[5613]`, source port `54272`.
- Auditd LOGIN: same PID `5613`, same timestamp, `auid=1000`, assigned `ses=54`, audit ID `1792593140.566:1414`.
- Investigation Workspace with Last 60 minutes and Audit session 54: candidate SSH/Auditd correlation displayed `audited_execs=77` and `monitored_file_ops=3`. These are aggregate result counts, not distinct typed-command counts.
- Selected session 54 file evidence: `/opt/soc-lab-test/soc003-kali-validation.txt` visible with successful monitored operations; first displayed audit ID `1792593148.187:1492`, original user 1000, session 54.
- Selected session 54 process evidence: `stat`, `chmod`, `id`, and `whoami`, user ID 1000, effective user ID 1000, audit session 54.
- SOC Overview also shows prior `soc003-validation.txt` file activity in session 53 and two failures followed by a success from `172.16.110.130`. This is separate local/self-SSH evidence and must not be attributed to the Kali session.
- Recent activity timeline is explicitly all sessions; nearby timeline events are not automatically session 54 evidence.

### Raw-event validation still required

Check all Auditd records sharing the same event ID before concluding the exact file operation:

```spl
index="soc-live" host="soc-ubuntu-01" source="/var/log/audit/audit.log"
| rex field=_raw "msg=audit\((?<audit_id>[^)]+)\)"
| search audit_id="1792593148.187:1492"
| rex field=_raw "type=(?<record_type>[A-Z_]+)"
| table _time audit_id record_type _raw
| sort 0 _time
```

Confirm the PATH filename and companion SYSCALL attributes `ses=54`, `auid=1000` and `success=yes`. Check additional audit IDs for later changes separately. The distinct event ID groups represent actions; several PATH/SYSCALL records for one ID are not several independent user operations.

After validation, write the final analyst case chronology, detection outcomes, attribution limits, and reasoned authorized-training disposition. Store sanitized screenshots in the repository.

## Stage 2 — trace the event across Splunk reports

Open the working three-tab [SOC dashboard definition](../../dashboards/soc-lab-dashboard-studio.json). Its saved-report data sources are already bound to the global time picker in the exported source. Set a narrow time window covering the session. On Investigation Workspace, enter the **newly observed Auditd session** in the `Audit session (case filter)` numeric input (the initial `33` is only a prior-session default, not the expected new session ID). Its candidate correlation and file/process evidence tables are session-scoped; the general activity timeline remains all sessions.

1. **Authentication:** In LINUX-DET-001, identify the new successful SSH connection using `socadmin`, `172.16.110.128`, `sshd[PID]`, port and UTC timestamp.
2. **Session binding:** In LINUX-COR-001, look for the matching Auditd `LOGIN` record with the same SSH PID on the same host and within the time bound. Extract its `ses` value. If this returns nothing, inspect `type=LOGIN` records manually and record correlation coverage limitations.
3. **Processes:** In LINUX-DET-004, narrow to that `ses` and review `whoami`, `id`, `bash`, `chmod`, `stat` with `auid`, `euid`, `pid`, `ppid`. Some expected login-initialization processes may have `euid=0`; do not assume each represents manual sudo use.
4. **File operations:** In LINUX-DET-003, identify `/opt/soc-lab-test/soc003-validation.txt`. Record `audit_id`, `PATH`, `SYSCALL`, operation success and the original login ID. One file operation may consist of several raw Auditd records.
5. **Timeline:** Review LINUX-INV-001 for chronology. Do not attribute a neighboring log event to this connection without PID/session evidence.
6. **Disposition:** Expected authorized laboratory test, unless observed evidence contradicts the planned sequence. Do not label this an actual enterprise intrusion.

## Stage 3 — targeted evidence queries

Find the newest SSH successes originating from Kali:

```spl
index="soc-live" host="soc-ubuntu-01" source="/var/log/auth.log"
"Accepted password" "172.16.110.128"
| rex field=_raw "sshd\[(?<ssh_pid>\d+)\]"
| table _time ssh_pid _raw
| sort 0 -_time
```

Locate the Auditd login records associated with an identified `ssh_pid` (replace the literal with the observed PID only after inspecting the SSH log):

```spl
index="soc-live" host="soc-ubuntu-01" source="/var/log/audit/audit.log"
"type=LOGIN"
| rex field=_raw "(?:^|\s)pid=(?<ssh_pid>\d+)"
| rex field=_raw "(?:^|\s)ses=(?<audit_session>\d+)"
| table _time ssh_pid audit_session _raw
| sort 0 -_time
```

Find all Auditd records containing the SOC-003 test file:

```spl
index="soc-live" host="soc-ubuntu-01" source="/var/log/audit/audit.log"
"soc003-validation.txt"
| rex field=_raw "msg=audit\((?<audit_id>[^)]+)\)"
| rex field=_raw "type=(?<record_type>[A-Z_]+)"
| table _time audit_id record_type _raw
| sort 0 _time
```

To see companion records with the same `audit_id`, run a second search for that exact value, not just the filename. Check `success=yes`, `nametype=CREATE` for creation, and session identity on associated SYSCALL records. Compare against the broader saved report.

## Expected deliverables (complete only after actual test)

- [ ] Timestamped Kali start / Ubuntu session.
- [x] Successful SSH log with source IP and PID.
- [x] Auditd LOGIN same PID and assigned session ID.
- [x] Session 54 process commands and original/effective user IDs recorded from dashboard.
- [ ] Verify raw grouped PATH/SYSCALL records for session 54 file events; dashboard file evidence and audit IDs captured.
- [ ] Optional lab HTTP test and response code.
- [ ] Screenshots of all three dashboard tabs with data and readable labels.
- [ ] Investigation timeline, conclusion, limitations, and analyst disposition.
- [x] Session-filtered Dashboard Studio JSON exported from the working configuration and committed before the SOC-003 exercise.

All outputs should be described as **controlled lab evidence**.
