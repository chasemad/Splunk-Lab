# SOC-001 — Suspicious authentication and privilege-related activity

## At a glance

**Investigation question:** When failed logins, account discovery, DNS activity and a group-membership change occur close together, what can an analyst reasonably conclude?

**What I did:** Examined **205 synthetic security events** in Splunk, reconstructed their sequence, and assessed whether the combined evidence warranted escalation.

**What it demonstrates:** SIEM searches, multi-source investigation, detection correlation, cautious interpretation of suspicious patterns, and concise reporting.

**Outcome:** The sequence justified **further investigation**, but the available synthetic records did **not** prove compromise or establish the actor responsible for the group change. This is a completed analysis exercise, not a live incident or an actual attack.

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

## Supporting evidence

The synthetic event sequence and assessment are documented above. Dedicated screenshots for the SOC-001 timeline and correlation searches have not yet been published. For authentic screenshots from the live Linux track, see the [SOC-003 evidence gallery](../SOC-003/case-report.md#evidence-gallery--real-screenshots).
