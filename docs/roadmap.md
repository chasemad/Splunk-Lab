# Project completion roadmap

The lab has three documented investigations, eight saved live Linux reports, a three-tab Splunk Dashboard Studio export, and published screenshot galleries. The work below separates completed capabilities from operational and validation gaps.

## Essential security and reliability

- [ ] Revoke or rotate the temporary HEC test token; document secure TLS certificate validation.
- [ ] Review forwarder credentials, file permissions, and TLS peer-verification settings without committing secrets.
- [ ] Make narrowly scoped Auditd rules persistent; verify them after a planned restart.
- [ ] Verify Universal Forwarder restart behavior, log rotation, and ingestion recovery.
- [ ] Check Ubuntu disk/LVM capacity, time synchronization, and Splunk retention constraints.

## Detection validation

- [ ] Export exact SPL for all eight saved Linux reports and compare it to repository versions.
- [x] Reconcile the HTTP query visible in the October 21 Splunk report-editor screenshot with the canonical GitHub SPL. Confirm **Save/reopen** persistence before treating exact saved-report parity as established.
- [x] Prepare a [validation matrix](linux-report-validation-matrix.md) and [test plan](detection-test-plan.md); runtime outcomes still need to be recorded.
- [ ] Revisit SSH failed-login threshold and cross-connection coverage using controlled fixtures.
- [ ] Review the two additional SOC-003 file-operation record groups if useful for deeper evidence completeness.
- [ ] Decide whether scheduled alerting is supported and warranted; validate before claiming alert coverage.

## Portfolio presentation

- [x] Publish the SOC-003 nine-image case gallery.
- [x] Publish the five-image Dashboard Studio gallery.
- [x] Improve the main README with direct links to strongest evidence.
- [x] Correct SOC-003 screenshot publication status.
- [x] Give the dashboard documentation a reader-focused introduction.
- [ ] Visually review every published PNG for readability, captions, redaction, and evidence integrity.
- [ ] Check remaining documentation links and claims; retain unresolved evidence gaps transparently.
- [x] Refine the [architecture diagram](../architecture/lab-setup.md) and prepare a [short walkthrough storyboard](recruiter-demo-script.md).
- [ ] Record and publish the actual demonstration video.

## Optional extensions

- [ ] Create justified MITRE ATT&CK mappings and a coverage/gap table.
- [ ] Add a small number of new, testable endpoint detections with analyst playbooks.
- [ ] Consider additional automation only after collection, detection, and validation are reliable.

## Existing evidence and reference documents

- [SOC-003 case report](../investigations/SOC-003/case-report.md)
- [Dashboard export and gallery](../dashboards/README.md)
- [Dashboard validation](dashboard-validation-2026-10-21.md)
- [Report inventory](report-save-runbook.md)
- [Detection catalog](../detections/linux/README.md)
- [Security limitations](limitations.md)

The SSH authentication baseline and three-failure candidate details belong in [LINUX-DET-001 documentation](../detections/linux/LINUX-DET-001-ssh-auth.md), not in this project-wide roadmap.
