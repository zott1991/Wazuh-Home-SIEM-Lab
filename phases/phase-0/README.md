# Phase 0 — Environment Setup & Recovery

**Status:** Complete

## Debrief Log

The original Phase 0/1 debrief is reproduced below for direct reading in GitHub.

---

## Phase 0 — Environment Setup & Recovery

# Wazuh Home Cybersecurity Lab — Debrief
Phase 0: Environment Setup, Recovery & Wazuh Deployment
Status: Complete — Wazuh Dashboard operational, ready for endpoint deployment.
# 1. Objective
Build an isolated home cybersecurity/SIEM lab using VirtualBox and Wazuh. The goal is to progress from infrastructure setup to endpoint telemetry, detection, investigation, controlled attack simulation, and security documentation suitable for a portfolio project.
# 2. Lab Architecture
Virtualization platform: Oracle VirtualBox
Wazuh: Official OVA, Wazuh 4.14.7
Wazuh VM: All-in-one deployment containing Wazuh Manager, Indexer, Dashboard, and supporting services
Wazuh VM networking: Adapter 1 = NAT; Adapter 2 = Host-only
Wazuh Host-only IP: 192.168.56.102
Wazuh NAT IP: 10.0.2.15
Private lab network: 192.168.56.0/24
Initial endpoint planned: Kali Linux VM
# 3. Kali Linux Account Recovery
The Kali VM account credentials were unavailable, so the account was recovered through GRUB recovery boot.
Recovery procedure:
Edit the GRUB Linux boot entry.
Change the root filesystem option from ro to rw.
Append init=/bin/bash to the Linux boot line.
Boot with Ctrl+X/F10 to obtain a root shell.
Remount the filesystem read/write: mount -o remount,rw /
Identify the normal user from /home or /etc/passwd.
Reset the account password with passwd <username>.
Run sync and reboot -f.
Lesson learned: Recovery mode may not provide a usable root shell when the root account is locked. Booting directly to a root shell with init=/bin/bash provides an alternate recovery path.
# 4. Wazuh OVA Deployment
The official Wazuh OVA was imported into VirtualBox.
Graphics controller was configured as VMSVGA.
Adapter 1 was configured as NAT to provide Internet access for updates and package operations.
Adapter 2 was configured as a VirtualBox Host-only Adapter to isolate security-lab traffic.
After booting, the Wazuh VM exposed the following addresses:
eth0 — 192.168.56.102/24 (Host-only)
eth1 — 10.0.2.15/24 (NAT)
# 5. Initial Dashboard Failure
The Wazuh dashboard initially returned a connection-refused/not-ready state. The dashboard service was found inactive and was enabled and started with systemd.
Commands used:
sudo systemctl enable wazuh-dashboard
sudo systemctl start wazuh-dashboard
The dashboard subsequently became reachable, but initially displayed a 'dashboard server is not ready yet' message. This led to investigation of the Wazuh Indexer and Manager.
# 6. Wazuh Indexer Troubleshooting
The indexer appeared to start under systemd, but HTTPS requests to port 9200 initially failed with a connection error.
Test:
curl -k https://127.0.0.1:9200
System resources were checked and did not indicate an obvious resource exhaustion problem:
Approximately 7.8 GiB RAM available to the VM.
4 virtual CPUs.
Approximately 11 GiB free on the 25 GiB root filesystem.
No swap configured.
The key evidence came from /var/log/wazuh-indexer/wazuh-cluster.log. The log showed the OpenSearch node successfully starting, binding to 127.0.0.1:9200, recovering indices, and cluster health changing from RED to GREEN. A transient security-plugin message indicated that the backend was not yet initialized, but there was no evidence of a permanent configuration failure.
# 7. Root Cause: systemd Startup Timeout
systemctl status showed the Wazuh Indexer failing with Result=timeout and status 143. The service was being terminated because systemd's default startup timeout was only 45 seconds, while the Java/OpenSearch indexer required longer to initialize.
A systemd drop-in override was created:
/etc/systemd/system/wazuh-indexer.service.d/override.conf
[Service]
TimeoutStartSec=5min
The override was verified with:
cat /etc/systemd/system/wazuh-indexer.service.d/override.conf
Then systemd was reloaded and the indexer restarted:
sudo systemctl daemon-reload
sudo systemctl restart wazuh-indexer
# 8. Successful Indexer Recovery
After increasing the startup timeout, the indexer remained active:
Active: active (running)
The decisive verification was:
curl -k https://127.0.0.1:9200
The response was: Unauthorized
This was a successful result. It confirmed that the HTTPS service on port 9200 was reachable and responding; the security layer was rejecting an unauthenticated request rather than the port being unavailable.
# 9. Dashboard Recovery
The Wazuh Dashboard service was restarted after the indexer became stable:
sudo systemctl restart wazuh-dashboard
The browser then successfully displayed the Wazuh login page at:
https://192.168.56.102
The Wazuh Overview page is now accessible. The dashboard currently reports that no agents are registered, which is expected for a clean initial deployment.
# 10. Troubleshooting Lessons Learned
Do not immediately reinstall a service that appears broken. Inspect service status, logs, ports, and system resources first.
A systemd Result=timeout does not necessarily mean the application itself failed. The application may simply need more startup time.
Application logs can reveal successful initialization even when systemd reports a failed service.
HTTP/HTTPS responses such as Unauthorized can be evidence that a service is healthy and reachable.
Keep the SIEM on a private Host-only network for the lab while retaining NAT separately for Internet access.
Document the troubleshooting process, not just the final configuration. Diagnosing the failure demonstrates more technical understanding than following installation steps alone.
# 11. Current State
VirtualBox lab environment operational.
Kali VM account recovered.
Wazuh 4.14.7 OVA operational.
Wazuh Manager investigated and operational.
Wazuh Indexer operational with a 5-minute systemd startup timeout override.
Wazuh Dashboard operational.
Host-only lab network operational at 192.168.56.0/24.
Wazuh Dashboard accessible from the Windows host.
No endpoints/agents registered yet.
# 12. Next Phase
