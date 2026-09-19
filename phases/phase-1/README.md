# Phase 1 — Wazuh Agent Deployment

**Status:** Complete

## Debrief Log

The Phase 1 portion of the original Phase 0/1 debrief is reproduced below for direct reading in GitHub.

---

Phase 1 will deploy the Kali Linux Wazuh Agent and establish the first monitored endpoint. After agent enrollment and connectivity are confirmed, the lab will move into telemetry collection, alert generation, threat hunting, MITRE ATT&CK mapping, controlled attack simulation, and incident investigation.
# 13. Portfolio Documentation Notes
Recommended evidence to capture throughout the project:
Network topology diagram.
VirtualBox VM configuration.
Wazuh Dashboard Overview.
Agent enrollment and connectivity.
Example security alerts.
Investigation timelines and relevant logs.
MITRE ATT&CK mappings.
Custom detection rules.
Controlled attack/test commands and resulting alerts.
Troubleshooting incidents and root-cause analysis.
Final architecture and lessons learned.

Debrief complete — Phase 0 closed.

# Phase 1 — First Endpoint Deployment
Status: COMPLETE — Kali Linux successfully enrolled and actively reporting to Wazuh.
# Objective
Deploy the first monitored endpoint by installing the Wazuh Agent on Kali Linux, enrolling it with the Wazuh Manager, validating network communication, and confirming the endpoint appears as active in the Wazuh Dashboard.
# Agent Configuration
Package: Linux DEB amd64
Wazuh Agent: 4.14.7
Agent name: kali-attacker
Group: default
Manager: 192.168.56.102
# Connectivity and Installation
Kali confirmed connectivity to the Wazuh Manager with 4/4 ICMP replies and 0% packet loss.
ping -c 4 192.168.56.102
The Wazuh Agent DEB package downloaded and installed successfully. The installed configuration was verified to contain the intended manager address and agent name.
grep -E '<address>|<agent_name>' /var/ossec/etc/ossec.conf
# Agent Service Validation
The agent was enabled and started. The first status check temporarily showed a reload state; a subsequent check showed the service as active and running. Core processes including wazuh-agentd, wazuh-execd, wazuh-syscheckd, wazuh-logcollector, and wazuh-modulesd were running.
# Network Topology Correction
The initial Dashboard view reported Kali at 10.0.2.15, the NAT address. Because the intended lab architecture uses a private Host-only network for Wazuh-to-endpoint traffic, a second VirtualBox adapter was added to Kali.
Kali eth1: 192.168.56.101/24
Wazuh Manager: 192.168.56.102/24
Route to 192.168.56.0/24: eth1
Kali-to-Wazuh ping: 4/4 replies, 0% packet loss
# Agent-to-Manager Verification
The Kali agent log confirmed a successful TCP connection to the Wazuh Manager on port 1514.
wazuh-agentd: INFO: (4102): Connected to the server ([192.168.56.102]:1514/tcp).
# Dashboard Verification
After refreshing the Endpoints page, Wazuh reported:
Agent ID: 001
Name: kali-attacker
IP: 192.168.56.101
OS: Kali GNU/Linux 2026.2
Version: v4.14.7
Status: active
Group: default
# Troubleshooting Lessons
Enrollment success does not by itself prove the endpoint is using the intended network path.
Validate topology independently with ip addr, ip route, ping, service status, and agent logs.
A temporary reloading state after a service reload is not automatically a failure; check the settled service state.
Compare Dashboard information with endpoint-side evidence when validating networking and enrollment.
# Evidence Captured
Agent installation screenshot.
Agent configuration screenshot.
Kali network interface screenshot.
Routing and ping screenshot.
Successful TCP/1514 connection screenshot.
Wazuh Endpoints screenshot showing kali-attacker as active at 192.168.56.101.
# Phase 1 Result
The first endpoint is successfully enrolled, connected, and actively reporting to the Wazuh Manager over the isolated Host-only lab network.
# Phase 2 — Telemetry & Detection
Next objective: establish a baseline of Kali telemetry, generate controlled security events, and investigate how Wazuh collects, correlates, alerts on, and presents those events.
# Phase 2 Roadmap
Verify baseline telemetry.
Inspect Wazuh event and alert views.
Generate safe, controlled authentication failures.
Trace an event from Kali logs to the Wazuh Dashboard.
Examine rule IDs, severity, and MITRE ATT&CK mappings.
Document the investigation workflow.
Later, create or tune a custom detection and validate it with a controlled test.
