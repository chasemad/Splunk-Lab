# SOC-001 — Suspicious authentication and privilege-related activity

**Dataset: synthetic. Status: investigation completed; escalated for further investigation.**

## Scope
205 synthetic CSV events: 45 VPN authentication, 80 endpoint process, 77 DNS, 3 identity audit. Splunk index: `soc_lab`. The CSV field `source_type` is not the built-in Splunk `sourcetype` metadata field.

## Entities
User `amorgan`, workstation `WS-017`, fictional documentation IP `203.0.113.77`, documentation-only domain `support-telemetry-check.example`, and lab group `Lab-Privileged-Operators`.

## Timeline (UTC)
| Time | Observed synthetic event |
|---|---|
| 14:08 | Last of eight failed VPN logins |
| 14:09 | Successful VPN login from same IP |
| 14:10 | `whoami.exe /groups` |
| 14:11 | PowerShell local Administrators group query |
| 14:12 | Unusual DNS query |
| 14:13 | Security group membership addition |
| 14:14 | `mmc.exe` execution |

Identity records also included Event IDs 4728 and 4729 and activity involving `Lab-Printer-Users`.

## Assessment
The tightly grouped authentication, discovery, DNS, and identity events justified escalation. **Compromise was not established.** Available evidence did not conclusively identify who initiated the group change or what privileges the group granted.

## Limitations and next evidence
Corroborate authenticating session, actor identity, group privileges, authorized change context and independent endpoint telemetry. The dataset is intentionally synthetic.

**[PLACEHOLDER: investigation timeline screenshot]**

**[PLACEHOLDER: detection correlation screenshot]**
