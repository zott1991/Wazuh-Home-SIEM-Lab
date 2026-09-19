# Phase 3 — Detection Development

**Status:** Complete

## Debrief Log

The complete Phase 3 debrief is reproduced below for direct reading in GitHub.

---

Wazuh Home SIEM Lab — Phase 3 Debrief
Detection Development, Validation, and Live Deployment
# 1. Phase Objective
The objective of Phase 3 was to move beyond basic SIEM telemetry validation and develop, test, deploy, and investigate a custom Wazuh detection. The phase focused on correlating repeated SSH attempts against nonexistent usernames from the same source IP within a defined time window.
# 2. Starting Environment
Wazuh version: 4.14.7
Wazuh server hostname: wazuh-server
Wazuh server host-only IP: 192.168.56.102
Kali agent name: kali-attacker
Kali agent ID: 001
Kali host-only IP: 192.168.56.101
Wazuh Manager, Indexer, and Dashboard were ultimately confirmed active.
# 3. Detection Concept
Before creating a custom rule, the existing Wazuh SSH detections were examined. Rule 5710 detects SSH attempts using a nonexistent user. Rule 2502 was also observed during repeated authentication failures and already provides a built-in Level 10 brute-force escalation.
Because Rule 2502 already handled repeated authentication failures, the custom detection was designed to correlate the more specific Rule 5710 event rather than simply duplicate the generic brute-force detection.
# 4. Telemetry Verification
A real Rule 5710 event was inspected in alerts.json. The decoded event contained:
• srcip: 192.168.56.101
• srcuser: fakeuser
• decoder: sshd
• location: journald
• Rule ID: 5710
• Rule level: 5
wazuh-logtest was then used to independently reproduce the same decoding and Rule 5710 match before the custom rule was created.
# 5. Custom Rule Implementation
A dedicated custom rules file was created at /var/ossec/etc/rules/100-ssh-enumeration.xml.
Final rule:
<group name="custom_sshd,">
  <rule id="100100" level="10" frequency="3" timeframe="120">
    <if_matched_sid>5710</if_matched_sid>
    <same_srcip />
    <description>Repeated SSH attempts against nonexistent usernames from the same source IP.</description>
    <mitre>
      <id>T1087</id>
    </mitre>
  </rule>
</group>
Rule logic: three Rule 5710 events from the same source IP within 120 seconds produce custom Rule 100100 at Level 10.
# 6. Pre-Deployment Testing
The custom rule was tested with wazuh-logtest before restarting the live Manager. Three SSH log events from the same source IP produced Rule 100100 with Level 10. The test confirmed that the decoder, source IP field, Rule 5710 parent event, frequency, timeframe, and same-source correlation all worked.
# 7. Infrastructure Issue and Recovery
During rule development, the Wazuh server filesystem filled completely. The 25 GB root filesystem reached 100% utilization, leaving approximately 20 KB free.
/var/ossec was approximately 21 GB.
/var/ossec/queue was approximately 20 GB.
/var/ossec/queue/vd/feed was approximately 11 GB and was left intact.
/var/ossec/queue/vd_updater/tmp was approximately 8.2 GB.
Temporary updater files under /var/ossec/queue/vd_updater/tmp/contents were removed.
After cleanup, approximately 8.2 GB was free and the filesystem returned to 68% utilization.
wazuh-manager, wazuh-indexer, and wazuh-dashboard were subsequently confirmed active.
This incident reinforced the importance of monitoring disk capacity in the lab. The vulnerability feed was intentionally left untouched; only temporary updater artifacts were removed.
# 8. Live Deployment
After pre-deployment validation, wazuh-manager was restarted to load the custom rule. The service returned to an active state. The complete Wazuh service set was then verified as active.
# 9. Controlled Detection Test
A controlled SSH test was performed from Kali against its own host-only address (192.168.56.101). A nonexistent account, fakeuser, was used and incorrect authentication was attempted three times before SSH disconnected.
The live Wazuh Manager generated Rule 100100. The resulting alert contained:
• Rule ID: 100100
• Level: 10
• Agent: kali-attacker (ID 001)
• Agent IP: 192.168.56.101
• Source IP: 192.168.56.101
• Source user: fakeuser
• Decoder: sshd
• Location: journald
• Frequency: 3
• MITRE mapping: T1087 / Account Discovery
The alert also preserved previous_output containing preceding SSH events, providing correlation context for the detection.
# 10. Dashboard Investigation
The custom alert was located in Wazuh Threat Hunting. The Events view displayed the kali-attacker event with Rule 100100, Level 10, and the custom description. The expanded Document Details view exposed the agent, source IP, source user, sshd decoder, journald location, frequency, MITRE mapping, full log, and previous correlated output.
# 11. Positive Validation
Positive test result: three matching Rule 5710 events from the same source IP within the 120-second timeframe caused Rule 100100 to fire at Level 10.
# 12. Negative Validation
A single synthetic Rule 5710 event was submitted to wazuh-logtest. The result matched Rule 5710 at Level 5, and Rule 100100 did not fire. This demonstrated that the custom correlation threshold was functioning rather than triggering on every individual nonexistent-user SSH event.
# 13. Important Detection Scope Note
The final rule detects repeated SSH attempts against nonexistent usernames from the same source IP. The controlled live test used the same nonexistent username (fakeuser) repeatedly. Therefore, the test demonstrates repeated nonexistent-user SSH activity, not proven multi-username account enumeration.
The rule is mapped to MITRE ATT&CK T1087 Account Discovery as an analytic mapping. The lab evidence demonstrates the observed event pattern; it does not establish attacker intent.
# 14. Evidence Retained
Custom rule file: /var/ossec/etc/rules/100-ssh-enumeration.xml
Pre-deployment wazuh-logtest output showing Rule 100100 firing.
Live alerts.json output for Rule 100100.
Pretty-printed JSON representation of the live alert.
Wazuh Threat Hunting Dashboard screenshot showing the custom alert.
Expanded Dashboard Document Details screenshot.
Negative wazuh-logtest result showing only Rule 5710.
Filesystem recovery evidence showing the temporary vulnerability updater data issue.
# 15. Lessons Learned
Inspect actual decoded telemetry before designing a correlation rule.
Use wazuh-logtest to validate custom rules before restarting services.
Avoid duplicating existing detections when designing custom analytics.
Test both positive and negative cases.
Distinguish observed telemetry from the analyst's interpretation of that telemetry.
Monitor disk usage on small SIEM lab VMs, especially when vulnerability feeds and updater data are enabled.
Preserve screenshots and raw alert data as evidence for portfolio documentation.
# 16. Phase 4 Roadmap / Candidate Next Steps
Create a second custom detection representing a different attack behavior.
Explore alert tuning to reduce duplicate or low-value events.
Investigate alert visualization and reporting workflows.
Add another endpoint to demonstrate multi-agent visibility.
Develop a small incident-response workflow around a detected event.
Address long-term storage and vulnerability-feed disk consumption before expanding the lab.
Organize the project evidence into a GitHub-friendly portfolio structure.
# 17. Phase 3 Outcome
Phase 3 successfully progressed the lab from SIEM telemetry validation into detection engineering. A custom Wazuh correlation rule was designed, tested, deployed, triggered by controlled activity, investigated in the Dashboard, and validated with both positive and negative tests.
