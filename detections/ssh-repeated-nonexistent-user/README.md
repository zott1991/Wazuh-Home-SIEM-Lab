# SSH Repeated Nonexistent-User Detection

**Rule ID:** `100100`  
**Severity:** Level 10  
**Parent:** `5710`  
**Frequency:** 3  
**Timeframe:** 120 seconds  
**Correlation:** Same source IP

This custom Wazuh rule correlates three Rule 5710 events from the same source IP within 120 seconds.

The detection was validated with `wazuh-logtest`, deployed to the live Wazuh Manager, triggered through controlled SSH activity, and verified in both `alerts.json` and the Wazuh Dashboard.

The controlled test used the same nonexistent username repeatedly, so the demonstrated behavior is repeated nonexistent-user SSH activity rather than proven multi-username account enumeration.
