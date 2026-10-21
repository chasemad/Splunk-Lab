# SOC-002 screenshot placeholders

The case report is at [investigations/SOC-002/README.md](../../investigations/SOC-002/README.md).

The following are **planned filenames**, not evidence already uploaded:

1. `01-ssh-authentication.png` — accepted SSH authentication for `sshd[4937]` from Kali.
2. `02-login-session-27.png` — Auditd LOGIN record identifying `ses=27`.
3. `03-session-processes.png` — `whoami`, `id`, `touch` recorded under session 27.
4. `04-file-creation-audit.png` — Audit event 894: `PATH` with `nametype=CREATE`, successful `SYSCALL`.
5. `05-investigation-timeline.png` — single chronological investigator view.

Before publishing an image, crop browser tabs, remove account identifiers and secrets, and verify that every visible log field is readable. Internal lab network IPs and hostnames are documented as a controlled virtual range. Use actual Splunk screenshots, not generated lookalikes.
