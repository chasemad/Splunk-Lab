# Historical correlation: COR-001

**Status:** saved report, validated on synthetic `soc_lab` data. Exact saved SPL export pending.

Correlates repeated failed VPN authentication, subsequent successful login, and subsequent PowerShell group discovery. Chronology was enforced with:

```spl
| where last_failure < first_success AND first_success < first_discovery
```

Observed: 8 failures (last 14:08 UTC), first success 14:09 UTC, first discovery 14:11 UTC for `amorgan`. This does not prove one underlying session or malicious intent. Username-only matching and fixed time windows are important limitations.

**Evidence status:** A dedicated screenshot of the historical COR-001 report has not been published. The result described above comes from the synthetic dataset; see the related [SOC-001 analysis](../../investigations/SOC-001/README.md).
