# Phase 3 — Detection Development

Phase 3 moved the lab from telemetry validation into custom detection engineering.

## Custom detection

**Rule:** `100100`  
**Severity:** Level 10  
**Parent rule:** `5710`  
**Frequency:** 3 events  
**Timeframe:** 120 seconds  
**Correlation:** Same source IP

The rule detects repeated SSH attempts against nonexistent usernames from the same source IP.

```text
Rule 5710
    ↓
3 matching events
    ↓
Same source IP
    ↓
Within 120 seconds
    ↓
Custom Rule 100100
    ↓
Level 10 Alert
```

The rule was:
1. Created as a custom Wazuh rule.
2. Tested with `wazuh-logtest`.
3. Loaded by the live Wazuh Manager.
4. Triggered through controlled SSH activity.
5. Verified in `alerts.json`.
6. Investigated in the Wazuh Dashboard.
7. Validated with positive and negative tests.

## Detection scope

The controlled live test repeatedly used the same nonexistent username (`fakeuser`). Therefore, the demonstrated behavior is repeated nonexistent-user SSH activity from one source, not proven multi-username account enumeration. The rule is mapped to MITRE ATT&CK `T1087` as an analytic mapping.

See the original debrief in `docs/original-debriefs/`.
