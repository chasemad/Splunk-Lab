# SOC-003 — Controlled Kali-to-Ubuntu Investigation

**State:** Investigation playbook prepared; no SOC-003 activity or evidence claimed complete yet.
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
- [ ] Successful SSH log with source IP and PID.
- [ ] Auditd LOGIN same PID and assigned session ID.
- [ ] Process commands and original/effective user IDs from that session.
- [ ] File create and modify operations and audit event IDs.
- [ ] Optional lab HTTP test and response code.
- [ ] Screenshots of all three dashboard tabs with data and readable labels.
- [ ] Investigation timeline, conclusion, limitations, and analyst disposition.
- [x] Session-filtered Dashboard Studio JSON exported from the working configuration and committed before the SOC-003 exercise.

All outputs should be described as **controlled lab evidence**.
