# Roadmap

- [x] Import 205 synthetic SOC events and investigate SOC-001.
- [x] Save four historical detection reports and one correlation report.
- [x] Install Ubuntu/Kali lab VMs and verify connectivity.
- [x] Connect ARM64 Universal Forwarder to Splunk Cloud.
- [x] Validate syslog, auth, Auditd and HTTP telemetry.
- [ ] Step 42: cleaner investigation timeline.
- [x] Step 43: SSH session attribution with Auditd (Kali SSH PID 4937 mapped to audit session 27).
- [x] Step 44: group related Auditd records by audit event ID.
- [x] Complete SOC-002 investigation and evidence-backed report (authorized lab activity).
- [ ] Build and validate Linux-specific detections (LINUX-DET-003 manual positive/negative tests passed for monitored lab file changes; saved Splunk Report operator-confirmed; broader tuning and scheduled-alert evaluation still pending).
- [ ] Build and validate Dashboard Studio dashboard.
- [ ] Export exact saved SPL searches.
- [ ] Capture and redact genuine screenshots.
- [ ] Verify Auditd rule persistence and forwarder/log rotation behavior.
- [ ] Investigate Ubuntu disk capacity and tune event volume.
- [ ] Rotate/revoke temporary HEC token and secure credential copies.
- [ ] Record short portfolio demonstration video.

See [detection-to-dashboard implementation plan](detection-to-dashboard-plan.md) for the exact order, test gates, report names and dashboard panels.

- [ ] Save prepared one-pass live report pack in Splunk Cloud (only LINUX-DET-003 confirmed saved so far).
- [ ] Build SOC-003 controlled Kali-to-Ubuntu investigation using validated reports and dashboard.

A corrected three-tab, edge-aligned grid definition was committed on 2026-10-21 after a reported gap validation error. Re-import this latest JSON before continuing with panel data validation.
