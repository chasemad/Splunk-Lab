# LINUX-DET-001 — SSH authentication activity

**Stage:** Exploratory baseline confirmed in Splunk; candidate multi-failure detection not yet validated or saved as a report.

## Observed activity (October 21, 2026)

The live Splunk search over `index="soc-live"` and Ubuntu's `/var/log/auth.log` returned **8 matching authentication events** in the selected window:

| Remote source | Successful logins | Failed logins |
|---|---:|---:|
| `172.16.110.128` (Kali lab VM) | 4 | 1 |
| `172.16.110.1` (VMware network address; originating system not yet established) | 3 | 0 |
| **Total** | **7** | **1** |

All eight extracted events referenced `socadmin` using password authentication. The failed attempt at **11:49:56.642 UTC** and successful attempt at **11:50:10.192 UTC** shared `sshd[5124]`, source IP `172.16.110.128`, and source port `52796` — a roughly **13.55-second** interval. A single failed password followed by a success is consistent with a possible mistyped password and **does not alone establish brute force or unauthorized access**.

## Files

- [LINUX-DET-001-ssh-session-summary.spl](LINUX-DET-001-ssh-session-summary.spl) — manually review failed/successful SSH events grouped by connection identifiers. **New improved query; run it before claiming its returned output was validated.**
- [LINUX-DET-001-ssh-failure-followed-success.spl](LINUX-DET-001-ssh-failure-followed-success.spl) — **candidate** threshold requiring three or more failures followed by success within one SSH connection. Current known eight-event sample should **not** satisfy this threshold (only one failure). This is not yet a tested alert.

## Validation and tuning checklist

1. Re-run the session summary query in Search & Reporting; confirm `sshd[5124]` has one failure and one success and compare actual field extraction to the raw log.
2. Examine `172.16.110.1` from the authorized host/network configuration rather than automatically trusting or labeling it malicious.
3. Run the candidate detection as an exploratory search. An empty result set is expected against the currently observed eight-event sample; do not claim full validation from a negative-only dataset.
4. Obtain safely documented positive multi-failure examples (or build controlled non-production fixture events) before evaluating sensitivity, thresholds, false positives and time windows. Do not generate uncontrolled authentication traffic.
5. Note differences between `Accepted password`, `Accepted publickey`, `Failed password`, `Failed publickey`, invalid-user messages and other SSH formats. Revisit extractions for real distribution.
6. For multi-connection failures from one source, design a separate user/IP-time-window correlation. The **same-PID-and-port** pilot intentionally does not capture attempts spread across multiple SSH sessions.
7. Save only a search tested for its stated purpose as a Splunk Report. Configure scheduling/alerting only after validated data volume, lag, permissions and duplicate-handling rules.

## Interpretation

- **Authentication success** is not proof of authorization.
- **One failure then success** is useful investigation context, not intrinsically a brute-force detection.
- **A shared SSH PID and source port** support the same SSH transport connection but do not independently establish what happened after authentication.
- **SSH source IP** and the actual user operating the remote machine are different questions; the later requires broader context.

## Evidence placeholders

- [ ] Sanitized screenshot: eight auth events and extracted fields.
- [ ] Sanitized screenshot: per-connection SSH session summary.
- [ ] Validated candidate query positive/negative test results.
- [ ] Full saved Splunk Report settings and SPL after review.
