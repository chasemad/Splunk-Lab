# Detection engineering catalog

The first five searches were saved as **Splunk reports**, not scheduled alerts. All operate on the **synthetic** `soc_lab` dataset. Exact SPL exports still need to be retrieved from Splunk before claiming these files reproduce saved reports verbatim.

| ID | Behavior | Verified outcome |
|---|---|---|
| DET-001 | Repeated failed VPN logins followed by success | Tuned out narrowly defined `svc_backup` from `10.10.9.10`; retained `amorgan` |
| DET-002 | PowerShell local/domain account and group discovery | Excluded ordinary `Get-Date`; identified group discovery |
| DET-003 | Windows 4728 privileged group additions | Found `amorgan` membership event; privileges not independently verified |
| DET-004 | Rare DNS query using `eventstats` | Identified `support-telemetry-check.example`; rarity is not maliciousness |
| COR-001 | Failures → success → group discovery | 8 failures, 1 success, 1 discovery; chronological conditions validated |

COR-001 uses the ordering predicate:

```spl
| where last_failure < first_success AND first_success < first_discovery
```

Known limitations: synthetic data, limited baseline, simplistic grouping, missing session identifiers, incomplete attribution. See [historical](historical/README.md), [correlations](correlations/README.md), and [planned Linux rules](linux/README.md).

**Evidence availability:** The historical report outcomes are documented above, but dedicated screenshots of these five synthetic-data reports have not been published. The [SOC-001 case](../investigations/SOC-001/README.md) provides the investigation context. For screenshot-backed live Linux evidence, start with [SOC-003](../investigations/SOC-003/case-report.md).
