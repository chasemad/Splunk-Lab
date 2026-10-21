# Lessons learned

1. Splunk index names matter: `soc-live` and `soc_live` are different.
2. A one-off HEC test with a certificate verification exception is not a secure sustained ingestion strategy; the lab transitioned to Universal Forwarder.
3. Distinguish source fields in imported CSVs (`source_type`) from Splunk metadata (`sourcetype`).
4. Tune narrow known-benign behaviors instead of broadly suppressing detections.
5. Rare DNS queries, privilege-group names and suspicious command strings require context, not automatic malicious labels.
6. Chronological correlation improves searches but does not prove session identity.
7. Auditd can emit several records for a single action; use its audit event identifier.
8. High syslog volume warrants review against storage and retention limits.

Add future troubleshooting notes with the symptom, evidence, root cause, correction, and validation.
