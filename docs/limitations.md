# Limitations and security considerations

- This is a personal controlled lab, not an enterprise production SOC.
- `soc_lab` contains synthetic events; `soc-live` contains actual lab telemetry. Do not conflate them.
- Saved historical searches are reports, not configured real-time alerts.
- Auditd records are not one-to-one with underlying operations.
- SSH events and process executions require verified session correlation; temporal proximity alone is insufficient.
- VMware NAT is not complete isolation; testing must remain restricted to owned VMs.
- The Python HTTP server is temporary and should run only when needed.
- Auditctl rules were temporary; persistence after reboot not yet verified.
- UF TLS connection was observed; full peer-verification configuration not separately audited.
- Temporary HEC test used `curl -k` because of certificate validation trouble; revoke/rotate that token and avoid TLS bypass in ongoing collection.
- Forwarder credential packages, tokens, raw sensitive logs and passwords must remain private.
- Ubuntu disk size/LVM layout, log rotation, file permissions, retention and time synchronization need further verification.
- Splunk Cloud trial limits and index retention may constrain results.
