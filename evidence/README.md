# Evidence gallery and validation checklist

Only upload genuine, reviewed, redacted evidence. Do not fabricate screenshots or expose credentials, tokens, sensitive logs, or private network details unnecessarily.

- [ ] Splunk Cloud active forwarder / destination (redacted)
- [x] Live source breakdown (`soc-live`) — [dashboard screenshot](../dashboards/screenshots/01-soc-operations-overview.png)
- [x] SSH accepted authentication from controlled Kali VM — [SOC-003 exhibit](../investigations/SOC-003/screenshots/01-kali-ssh-authentication.png)
- [x] Auditd file-change event and grouped audit ID — [five raw records](../investigations/SOC-003/screenshots/06-raw-file-creation-audit-id.png)
- [x] Auditd process execution summary — [session 54](../investigations/SOC-003/screenshots/05-session54-process-and-file-evidence.png)
- [x] HTTP 200/404 request results — [request table](../dashboards/screenshots/05-clean-http-request-view.png)
- [ ] Five historical saved reports
- [ ] COR-001 ordered event sequence
- [ ] SOC-001 timeline
- [ ] SOC-002 case is documented; dedicated sanitized screenshots not yet published
- [ ] LINUX-DET-003 exploratory grouping: 25 raw events to 7 audit operations; screenshot sanitized/upload pending
- [ ] Linux detections validation (positive/negative tests and tuning still pending)
- [x] Dashboard Studio gallery published — [five screenshots](../dashboards/README.md#screenshot-gallery)

Record query, time range, UTC/local time basis, source and what each screenshot actually proves.

## Published evidence collections

- [SOC-003 analyst evidence gallery](../investigations/SOC-003/case-report.md#evidence-gallery--real-screenshots) — nine authentic screenshots, six core exhibits and three context/preflight images.
- [Dashboard Studio image gallery](../dashboards/README.md#screenshot-gallery) — five dashboard, session-filter and HTTP data-quality images.

Edits are limited to cropping, framing, and external explanatory captions. The Splunk interface and evidence values have not been recreated. Case session 53 is not attributed to Kali session 54.
