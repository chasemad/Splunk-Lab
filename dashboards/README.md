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
- [ ] Finish functional validation of the working three-tab Dashboard Studio dashboard (all report names resolve; native global time filtering configured for all eight, selected panels tested; HTTP and session-attribution checks pending).
- [ ] Export exact saved SPL searches.
- [ ] Capture and redact genuine screenshots.
- [ ] Verify Auditd rule persistence and forwarder/log rotation behavior.
- [ ] Investigate Ubuntu disk capacity and tune event volume.
- [ ] Rotate/revoke temporary HEC token and secure credential copies.
- [ ] Record short portfolio demonstration video.

See [detection-to-dashboard implementation plan](detection-to-dashboard-plan.md) for the exact order, test gates, report names and dashboard panels.

- [x] Save eight live Linux reports in Splunk Cloud (operator-confirmed 2026-10-21; individual results, app contexts and permissions still require validation).
- [ ] Build SOC-003 controlled Kali-to-Ubuntu investigation using validated reports and dashboard.

The latest [production JSON export](soc-lab-dashboard-studio.json) was copied directly from the operator's working Splunk Cloud Dashboard Studio source on 2026-10-21. It preserves all three edge-aligned Grid tabs and binds each of the eight saved report sources to the global time picker. See [source setup/troubleshooting](dashboard-studio-setup.md) and [validation findings](../docs/dashboard-validation-2026-10-21.md). The [temporary diagnostic JSON](diagnostic-dashboard-studio.json) is retained for reproducible troubleshooting.

## Latest validation checkpoint

The three-tab dashboard displays live panels without missing-object messages in the screenshots from 2026-10-21. The current data totals reconcile, but saved-report time scope, an unparsed HTTP diagnostic row, and attribution of file evidence to the selected SSH session still need testing. See [dashboard validation findings](../docs/dashboard-validation-2026-10-21.md).

- [x] Sync actual working Dashboard Studio JSON export (all eight named saved reports and global time bindings) to GitHub.
