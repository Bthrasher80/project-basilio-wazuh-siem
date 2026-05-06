Project Basilio — Wazuh SIEM Deployment
Overview
This project documents the deployment and configuration of a Wazuh SIEM environment on a Ubuntu 22.04 LTS VM running on Proxmox as part of Project Basilio, a self-built SOC home lab. The deployment uses Docker Compose to run Wazuh Manager, Indexer, and Dashboard as a single-node stack. The lab demonstrates real-world troubleshooting, agent deployment, and active threat detection mapped to MITRE ATT&CK.
Environment: Ubuntu 22.04 LTS VM (wazuh-server) hosted on Proxmox
Wazuh Version: v4.7.2 via Docker
Host IP: 192.168.1.170
Stack: Wazuh Manager + Wazuh Indexer + Wazuh Dashboard (single-node Docker deployment)
Objectives

Deploy Wazuh SIEM as a containerized single-node stack on a Proxmox-hosted Ubuntu VM
Register and verify a Wazuh agent reporting to the manager
Confirm security event detection and MITRE ATT&CK mapping is operational
Simulate privilege escalation activity and validate alert generation

Deployment Steps
1. Docker Runtime Installation and Verification
Installed Docker CE and Docker Compose on the Ubuntu VM. Enabled Docker as a systemd service and confirmed versions: Docker 29.2.1, Docker Compose v5.0.2.
📸 screenshots/PB-WAZUH-01_Docker_Runtime_Installed_and_Verified.png
2. Repository Clone and Deployment Troubleshooting
Cloned wazuh/wazuh-docker and attempted single-node deployment. Resolved two blocking issues before successful deployment.
Issue 1 — Image Tag 5.0.0 Not Found: The repo referenced wazuh/wazuh-indexer:5.0.0 which did not exist in the Docker registry at time of deployment. Resolved by pinning all image tags to v4.7.2 in docker-compose.yml.
📸 screenshots/PB-WAZUH-02_Image_Tag_Resolution_Failure_5_0_0.png
Issue 2 — Port 443 Already in Use: A native wazuh-dashboard service was binding port 443, preventing the container from starting. Resolved by stopping and disabling the conflicting service, confirming port 443 was free with ss -tulnp, then redeploying cleanly.
📸 screenshots/PB-WAZUH-02_Port_443_Conflict_Resolution.png
3. Successful Stack Deployment
After resolving all conflicts, docker compose up -d completed successfully with all components running: single-node-wazuh.indexer-1, single-node-wazuh.manager-1, and single-node-wazuh.dashboard-1. A Port 1514 conflict was also encountered and resolved during a restart cycle prior to final clean deployment.
📸 screenshots/PB-WAZUH-04_Port_1514_Manager_Bind_Conflict.png
4. Dashboard Access Confirmed
Accessed the Wazuh Dashboard via browser at https://192.168.1.170. Login page confirmed the stack was reachable and serving the web interface.
📸 screenshots/PB-WAZUH-03_Dashboard_Login_Page.png
5. Dashboard Overview — Agent Active
Logged into the dashboard. Confirmed 1 total agent, 1 active, 0 disconnected. All Wazuh modules confirmed visible including Security Events, Integrity Monitoring, MITRE ATT&CK, Vulnerabilities, PCI DSS, and NIST 800-53.
📸 screenshots/PB-WAZUH-04_Dashboard_Overview_Agent_Active.png
6. Agent Registration Confirmed
IDNameIPOSVersionStatus001wazuh-server127.0.0.1Ubuntu 22.04.5 LTSv4.7.2Active
Agent coverage: 100%
📸 screenshots/PB-WAZUH-06_Agents_Overview_Server_Active.png
7. Agent Service Health Verified
Confirmed wazuh-agent.service active and running with all five processes healthy: wazuh-execd, wazuh-agentd, wazuh-syscheckd, wazuh-logcollector, wazuh-modulesd.
📸 screenshots/PB-WAZUH-07_Agent_Service_Status_Running.png
Threat Detection Validation
8. Security Events Dashboard
Confirmed active alert generation in the Security Events module. 430 total alerts in the last 24 hours. MITRE ATT&CK wheel showing active detections: Sudo and Sudo Caching, Valid Accounts, Disable or Modify Tools.
📸 screenshots/PB-WAZUH-08_Security_Events_Dashboard_Privilege_Escalation.png
9. Privilege Escalation Detected — MITRE Mapping Confirmed
TechniqueTacticDescriptionRule IDT1548.003Privilege Escalation, Defense EvasionSuccessful sudo to ROOT executed5402T1078Defense Evasion, Persistence, Privilege Escalation, Initial AccessPAM: Login session opened5501
📸 screenshots/PB-WAZUH-09_Privilege_Escalation_Detected_MITRE.png
10. Sensitive File Access — Shadow File Read
Ran sudo cat /etc/shadow to simulate post-compromise credential harvesting. Validates that Wazuh audit monitoring was active and logging root-level sensitive file access.
📸 screenshots/PB-WAZUH-10_Root_Access_Shadow_File_Read.png
MITRE ATT&CK Coverage
Technique IDNameTactic(s)T1548.003Abuse Elevation Control Mechanism: Sudo and Sudo CachingPrivilege Escalation, Defense EvasionT1078Valid AccountsDefense Evasion, Persistence, Privilege Escalation, Initial Access
Troubleshooting Log
IssueRoot CauseResolutionImage tag 5.0.0 not foundNon-existent tag referenced in docker-compose.ymlPinned all images to v4.7.2Port 443 conflictNative wazuh-dashboard service binding the portStopped and disabled native service, freed portPort 1514 conflictManager port conflict on restart cycleFull stack down, port verified free, redeployed
Key Takeaways

Wazuh v4.7.2 single-node Docker deployment is stable on a Proxmox-hosted Ubuntu 22.04 VM
Port conflicts are common when a native Wazuh install coexists with a Docker deployment — always verify with ss -tulnp before deploying
Wazuh maps privilege escalation and sensitive file access to MITRE ATT&CK out of the box with no custom rules required
Shadow file read maps to real-world credential access behavior (T1003)

Related Projects

Project Basilio — Main Hub
Network Segmentation
Centralized Logging
TOR Browser Threat Hunt
