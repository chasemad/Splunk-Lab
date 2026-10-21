# Splunk SOC Lab — Project demonstration

This project can be explored **now**, without a video. The strongest walkthrough is the completed SOC-003 investigation, supported by real screenshots from Splunk Cloud.

## Start with the evidence

1. **[SOC-003 case study](../investigations/SOC-003/case-report.md)** — Follow an authorized SSH login from Kali to Ubuntu, confirm the matching Linux audit session, and examine process and file evidence. Includes **nine real screenshots**.
2. **[Three-tab Splunk dashboard](../dashboards/README.md)** — Explore the SOC Overview, Endpoint Activity and Investigation Workspace, with **five screenshots** and the exported dashboard definition.
3. **[Evidence gallery](../evidence/README.md)** — See the main exhibits in one place.
4. **[SOC-002 investigation](../investigations/SOC-002/README.md)** — Read an earlier live-log correlation case.
5. **[SOC-001 investigation](../investigations/SOC-001/README.md)** — Review a separate analysis using a synthetic security dataset.

## Planned short video

A recorded walkthrough has **not yet been published**. The [2–4 minute storyboard and narration](../docs/recruiter-demo-script.md) are prepared for when the lab recording is produced. The proposed sequence is:

- Introduce the lab purpose and log collection.
- Show the actual dashboard and monitored data sources.
- Walk through the SOC-003 SSH authentication and Auditd session correlation.
- Explain the raw file-creation evidence and case disposition.
- Close with the engineering lessons and limits of the exercise.

Any future recording should use authentic, appropriately redacted Splunk evidence and avoid implying that authorized tests were real intrusions or that saved reports were production alerts.
