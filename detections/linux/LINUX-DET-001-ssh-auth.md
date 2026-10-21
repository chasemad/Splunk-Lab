# LINUX-DET-001 — SSH authentication activity

**Stage:** Authentication event extraction and per-connection session summary validated in Splunk; the three-failure detection remains a candidate and is now operator-confirmed saved as a **session-summary report**. The stricter threshold remains an experimental, unscheduled candidate.

## Observed activity (October 21, 2026)

The live Splunk search over `index="soc-live"` and Ubuntu's `/var/log/auth.log` returned **8 matching authentication events** in the selected window:

| Remote source | Successful logins | Failed logins |
|---|---:|---:|
| `172.16.110.128` (Kali lab VM) | 4 | 1 |
| `172.16.110.1` (VMware network address; originating system not yet established) | 3 | 0 |
| **Total** | **7** | **1** |

All eight extracted events referenced `socadmin` using password authentication. The failed attempt at **11:49:56.642 UTC** and successful attempt at **11:50:10.192 UTC** shared `sshd[5124]`, source IP `172.16.110.128`, and source port `52796` — a roughly **13.55-second** interval. A single failed password followed by a success is consistent with a possible mistyped password and **does not alone establish brute force or unauthorized access**.

## Files

- [LINUX-DET-001-ssh-session-summary.spl](LINUX-DET-001-ssh-session-summary.spl) — manually review failed/successful SSH events grouped by connection identifiers. The grouped session summary was tested against the eight-event baseline; saved SPL parity remains pending.
- [LINUX-DET-001-ssh-failure-followed-success.spl](LINUX-DET-001-ssh-failure-followed-success.spl) — **candidate** threshold requiring three or more failures followed by success within one SSH connection. Current known eight-event sample should **not** satisfy this threshold (only one failure). This is not yet a tested alert.

## Confirmed session-summary validation (2026-10-21)

The grouped [session summary SPL](LINUX-DET-001-ssh-session-summary.spl) was run in Splunk Search & Reporting. Eight matching authentication records were reduced to seven connections in the results table. Six connections had one successful login and no failures; connection `sshd[5124]`, source `172.16.110.128:52796`, recorded one failure then one successful authentication. The query calculated `time_between=13.55` seconds. This corroborates the individual records, but does not establish malicious activity.

**Still pending:** Qualifying live-event and cross-connection validation, threshold tuning, saved-SPL parity, and alert scheduling decision. A separate in-memory synthetic positive test passed on 2026-10-21.

## Candidate threshold negative test — 2026-10-21

The three-or-more-failures-before-success [candidate SPL](LINUX-DET-001-ssh-failure-followed-success.spl) was run in Splunk Search & Reporting with a time range containing the eight known SSH authentication records. Splunk displayed **8 matching input events** and **Statistics (0)** after aggregation and the threshold filter. This is consistent with the known single-failure-then-success connection (`sshd[5124]`) and six single-success connections.

**Interpretation:** The known one-failure sequence is correctly excluded by the three-failure threshold. This is negative-case behavior only: a qualifying positive case, broader cross-connection detection, threshold assessment, and report/alert setup remain untested. The 8 input log events are not eight detections.

## Synthetic positive test — 2026-10-21

A controlled in-memory `makeresults` search generated four **synthetic**, non-indexed SSH authentication records (three failed then one accepted). Splunk returned one matching group with account `socadmin`, source `192.0.2.10`, **3 failures**, and **1 success**. This confirms the tested parsing and threshold logic on that fixture, not broad real-world detection effectiveness; `192.0.2.10` is a documentation-only address, not a real remote lab client.

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
- [x] Candidate query positive/negative outcomes described in this document (synthetic positive and known-data negative); screenshots not yet published.
- [ ] Full saved Splunk Report settings and SPL after review.
