# Live telemetry inventory

All sources below were observed in `index="soc-live"` with host `soc-ubuntu-01`.

| File | Sourcetype | Validation |
|---|---|---|
| /var/log/syslog | syslog | `logger -t soc-lab-test "SOC-LIVE-AUTO-001 Automatic system log forwarding test"` found in Splunk |
| /var/log/auth.log | linux_secure | Kali SSH accepted-password and session events |
| /var/log/audit/audit.log | linux:audit | File-change and process execution Auditd events |
| /opt/soc-web-logs/access.log | soc:web:access | HTTP GET 200 and 404 from controlled test server |

Snapshot before HTTP addition: syslog 5,263, audit 644, auth 221 = 6,128 records. This is not a current total.

## Validation SPL

```spl
index="soc-live" host="soc-ubuntu-01"
| stats count by source sourcetype
| sort - count
```

Process summary previously showed 167 matching syscall records and 58 distinct process names. These are not necessarily 167 unique actions.

See the [published SOC Overview screenshot](../dashboards/screenshots/01-soc-operations-overview.png) for the live source inventory.

See [SOC-003 evidence](../investigations/SOC-003/case-report.md#evidence-gallery--real-screenshots) for authentication and Auditd records, and the [dashboard HTTP gallery](../dashboards/README.md#screenshot-gallery) for web-request parsing.
