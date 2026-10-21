# Auditd lab rules

Auditd was active with `enabled 1` and `lost 0` at validation time. The following **temporary** auditctl rules were applied in the lab; persistence is not verified.

## File changes

```bash
sudo auditctl -a always,exit \
  -F arch=b64 \
  -F dir=/opt/soc-lab-test \
  -F perm=wa \
  -F key=soc_lab_file_change
```

## Selected process execution

```bash
sudo auditctl -a always,exit \
  -F arch=b64 \
  -S execve \
  -S execveat \
  -F auid=1000 \
  -k soc_lab_exec
```

Auditd emits SYSCALL, PATH, CWD, PROCTITLE and other records for a single underlying operation. Use the audit event ID from `msg=audit(...)` to group related records; do not count each record as a unique user action.

Next: document narrowly scoped persistent rules, test after reboot, and monitor log volume.

**[PLACEHOLDER: Auditd file-change and process screenshots]**
