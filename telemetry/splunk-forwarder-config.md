# Splunk Universal Forwarder

Verified version: 10.4.3 ARM64, installed under `/opt/splunkforwarder` and running with dedicated `splunkfwd` account.

The monitored file configuration was placed at `/opt/splunkforwarder/etc/apps/soc_lab_inputs/local/inputs.conf`. This is a sanitized equivalent, not an exported configuration file.

```ini
[monitor:///var/log/auth.log]
disabled = 0
index = soc-live
sourcetype = linux_secure

[monitor:///var/log/syslog]
disabled = 0
index = soc-live
sourcetype = syslog

[monitor:///var/log/audit/audit.log]
disabled = 0
index = soc-live
sourcetype = linux:audit

[monitor:///opt/soc-web-logs/access.log]
disabled = 0
index = soc-live
sourcetype = soc:web:access
```

The Splunk Cloud UF credentials package was installed privately; the forwarder showed an active SSL destination on port 9997. Do not commit the `.spl` package or any tokens. Confirm file permissions, effective settings, TLS verification, log rotation and persistence after reboot.

**Evidence status:** [SOC-003 Exhibit 09](../investigations/SOC-003/case-report.md#09--ubuntu-monitoring-preflight) includes a point-in-time forwarder service preflight. A dedicated, sanitized forward-server configuration screenshot has not been published. Do not commit actual deployment credentials or tokens.
