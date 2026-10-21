# Historical detection reports (synthetic)

DET-001 through DET-004 are saved in Splunk Cloud and were validated against `index=soc_lab`. The handoff records objectives and outcomes but not full SPL definitions. Export each report's exact SPL before adding a `.spl` file.

- **DET-001**: failed VPN attempts followed by success; tuning exclusion `| where NOT (user="svc_backup" AND src_ip="10.10.9.10")`.
- **DET-002**: selected PowerShell `Get-LocalGroupMember`, `Get-ADGroupMember`, `Get-LocalUser` discovery.
- **DET-003**: Windows Event ID 4728 and group names containing `privileged`.
- **DET-004**: rare DNS lookups using `eventstats`; flagged `support-telemetry-check.example`.

For each future exported rule, include source requirements, complete SPL, time range, expected result, false positives, tuning rationale and limitations.

**[PLACEHOLDER: DET-001–004 screenshots]**
