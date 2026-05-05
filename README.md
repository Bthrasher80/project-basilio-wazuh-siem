# Project Basilio — Wazuh SIEM Deployment

## Overview

This project documents the deployment and configuration of a Wazuh SIEM environment on a physical Ubuntu 22.04 server as part of Project Basilio, a self-built SOC home lab. The deployment uses Docker Compose to run Wazuh Manager, Indexer, and Dashboard as a single-node stack. The lab demonstrates real-world troubleshooting, agent deployment, and active threat detection mapped to MITRE ATT&CK.

**Environment:** Physical Ubuntu 22.04 LTS server (`wazuh-server`) running Wazuh v4.7.2 via Docker  
**Host IP:** 192.168.1.170  
**Stack:** Wazuh Manager + Wazuh Indexer + Wazuh Dashboard (single-node Docker deployment)

---

## Objectives

- Deploy Wazuh SIEM as a containerized single-node stack on bare metal Ubuntu
- Register and verify a Wazuh agent reporting to the manager
- Confirm security event detection and MITRE ATT&CK mapping is operational
- Simulate privilege escalation activity and validate alert generation

---

## Deployment Steps

### 1. Docker Runtime Installation and Verification
Installed Docker CE and Docker Compose on the Ubuntu host. Enabled Docker as a systemd service and confirmed versions: Docker 29.2.1, Docker Compose v5.0.2.

📸 `screenshots/PB-WAZUH-01_Docker_Runtime_Installed_and_Verified.png`

---

### 2. Repository Clone and Deployment Troubleshooting
Cloned `wazuh/wazuh-docker` and attempted single-node deployment. Resolved two blocking issues:

**Issue 1 — Image Tag 5.0.0 Not Found:** Repo referenced a non-existent image tag. Resolved by pinning all images to v4.7.2 in docker-compose.yml.

📸 `screenshots/PB-WAZUH-02_Image_Tag_Resolution_Failure_5_0_0.png`

**Issue 2 — Port 443 Already in Use:** Native wazuh-dashboard service was binding port 443. Resolved by stopping the conflicting service and confirming the port was free with `ss -tulnp`.

📸 `screenshots/PB-WAZUH-02_Port_443_Conflict_Resolution.png`

---

### 3. Successful Stack Deployment
All three containers deployed successfully: wazuh.indexer-1, wazuh.manager-1, wazuh.dashboard-1. A Port 1514 conflict was also resolved during a restart cycle prior to final clean deployment.

📸 `screenshots/PB-WAZUH-04_Port_1514_Manager_Bind_Conflict.png`

---

### 4. Dashboard Access Confirmed
Accessed Wazuh Dashboard at `https://192.168.1.170`. Login page confirmed stack was serving the web interface.

📸 `screenshots/PB-WAZUH-03_Dashboard_Login_Page.png`

---

### 5. Dashboard Overview — Agent Active
Confirmed 1 total agent, 1 active, 0 disconnected. All modules visible including Security Events, MITRE ATT&CK, Vulnerabilities, PCI DSS, and NIST 800-53.

📸 `screenshots/PB-WAZUH-04_Dashboard_Overview_Agent_Active.png`

---

### 6. Agent Registration Confirmed

| ID | Name | IP | OS | Version | Status |
|----|------|----|----|---------|--------|
| 001 | wazuh-server | 127.0.0.1 | Ubuntu 22.04.5 LTS | v4.7.2 | Active |

📸 `screenshots/PB-WAZUH-06_Agents_Overview_Server_Active.png`

---

### 7. Agent Service Health Verified
Confirmed wazuh-agent.service active/running with all five processes healthy: wazuh-execd, wazuh-agentd, wazuh-syscheckd, wazuh-logcollector, wazuh-modulesd.

📸 `screenshots/PB-WAZUH-07_Agent_Service_Status_Running.png`

---

## Threat Detection Validation

### 8. Security Events Dashboard
430 total alerts generated in 24 hours. MITRE ATT&CK wheel showing active detections: Sudo and Sudo Caching, Valid Accounts, Disable or Modify Tools.

📸 `screenshots/PB-WAZUH-08_Security_Events_Dashboard_Privilege_Escalation.png`

---

### 9. Privilege Escalation Detected — MITRE Mapping Confirmed

| Technique | Tactic | Description | Rule ID |
|-----------|--------|-------------|---------|
| T1548.003 | Privilege Escalation, Defense Evasion | Successful sudo to ROOT executed | 5402 |
| T1078 | Defense Evasion, Persistence, Privilege Escalation, Initial Access | PAM: Login session opened | 5501 |

📸 `screenshots/PB-WAZUH-09_Privilege_Escalation_Detected_MITRE.png`

---

### 10. Sensitive File Access — Shadow File Read
Ran `sudo cat /etc/shadow` to simulate post-compromise credential harvesting. Validates Wazuh audit monitoring was active and logging root-level sensitive file access.

📸 `screenshots/PB-WAZUH-10_Root_Access_Shadow_File_Read.png`

---

## MITRE ATT&CK Coverage

| Technique ID | Name | Tactic(s) |
|---|---|---|
| T1548.003 | Sudo and Sudo Caching | Privilege Escalation, Defense Evasion |
| T1078 | Valid Accounts | Defense Evasion, Persistence, Privilege Escalation, Initial Access |

---

## Troubleshooting Log

| Issue | Root Cause | Resolution |
|---|---|---|
| Image tag 5.0.0 not found | Non-existent tag in docker-compose.yml | Pinned all images to v4.7.2 |
| Port 443 conflict | Native wazuh-dashboard binding the port | Stopped native service, freed port |
| Port 1514 conflict | Manager port conflict on restart | Full stack down, port verified free, redeployed |

---

## Key Takeaways

- Wazuh v4.7.2 single-node Docker is stable on bare metal Ubuntu 22.04
- Port conflicts are common when native Wazuh coexists with Docker — always verify with `ss -tulnp`
- Wazuh maps privilege escalation and sensitive file access to MITRE ATT&CK out of the box
- Shadow file read maps to real-world credential access behavior (T1003)

---

## Related Projects

- [Project Basilio — Main Hub](https://github.com/Bthrasher80/project-basilio)
- [Network Segmentation](https://github.com/Bthrasher80/project-basilio-network-segmentation)
- [Centralized Logging](https://github.com/Bthrasher80/project-basilio-centralized-logging)
- [TOR Browser Threat Hunt](https://github.com/Bthrasher80/project-basilio-tor-threat-hunt)
