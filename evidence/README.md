# Evidence capture checklist

Only upload genuine, reviewed, redacted evidence. Do not fabricate screenshots or expose credentials, tokens, sensitive logs, or private network details unnecessarily.

- [ ] Splunk Cloud active forwarder / destination (redacted)
- [ ] Live source breakdown (`soc-live`)
- [ ] SSH accepted authentication from controlled Kali VM
- [ ] Auditd file-change event and grouped audit ID
- [ ] Auditd process execution summary
- [ ] HTTP 200/404 request results
- [ ] Five historical saved reports
- [ ] COR-001 ordered event sequence
- [ ] SOC-001 timeline
- [x] SOC-002 verified timeline and attribution reviewed in conversation; sanitized GitHub screenshots still pending
- [ ] LINUX-DET-003 exploratory grouping: 25 raw events to 7 audit operations; screenshot sanitized/upload pending
- [ ] Linux detections validation (positive/negative tests and tuning still pending)
- [ ] Dashboard Studio screenshot (pending)

Record query, time range, UTC/local time basis, source and what each screenshot actually proves.
