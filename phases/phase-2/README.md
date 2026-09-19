# Phase 2 — SIEM Telemetry Validation

Phase 2 validated the end-to-end telemetry pipeline using controlled SSH authentication activity.

## Test
A controlled SSH authentication failure was generated on Kali and collected through journald by the Wazuh Agent.

Wazuh produced:
- Rule `5710` — SSH attempt using a nonexistent user
- Rule `5503` — PAM login failure
- Rule `2502` — repeated password failures / brute-force escalation

## Investigation
The resulting telemetry was verified in `alerts.json` and investigated through Wazuh Threat Hunting.

See the original debrief in `docs/original-debriefs/`.
