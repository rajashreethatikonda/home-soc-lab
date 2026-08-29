# Home SOC Lab — Simulated Attack Detection with Splunk

## Overview
Built a home SOC lab to simulate a real-world cyberattack and detect it using a SIEM. An attacker machine (Kali Linux) launches a brute force attack against a Windows 10 victim machine. Splunk ingests Windows Event Logs and generates detection alerts.

## Lab Architecture

Kali Linux (Attacker) ──── Host-Only Network ──── Windows 10 (Victim + Splunk SIEM)
192.168.56.102                                      192.168.56.101

## Tools Used

| Tool | Purpose |
|---|---|
| VirtualBox | VM hypervisor |
| Kali Linux 2026.2 | Attacker machine |
| Windows 10 Pro | Victim machine and SIEM host |
| Splunk Enterprise 10.4 | Log ingestion, detection, dashboards |
| Splunk Universal Forwarder | Windows Event Log shipping to Splunk |
| Hydra v9.7 | RDP brute force simulation |

## Attack Simulations

### 1. RDP Brute Force — MITRE ATT&CK T1110
- Hydra launched from Kali Linux targeting testuser account via RDP port 3389
- Generated 78+ failed logon events (Windows Event ID 4625)

### 2. Suspicious PowerShell Execution — MITRE ATT&CK T1059.001
- Encoded PowerShell command executed on victim machine
- Generated 56 process creation events (Windows Event ID 4688)

## Detections Built in Splunk

### Alert 1 — Brute Force Detection (Severity: Medium)

    index=main EventCode=4625 | stats count by host | where count > 5

Triggers when failed logins exceed 5 within a scheduled window.

### Alert 2 — Suspicious PowerShell Execution (Severity: High)

    index=main EventCode=4688

Triggers on any encoded PowerShell process creation event.

## SOC Dashboard
Built a real-time Splunk dashboard with two panels:
- Failed Login Attempts Over Time — clear spike visible during Hydra attack
- Process Creation Events Over Time — spikes during PowerShell simulation

![Dashboard](https://github.com/user-attachments/assets/44afc338-7a44-40cb-a4d2-41d896b33d32)

## Incident Summary

| Field | Value |
|---|---|
| Date | 22 August 2026 |
| Attacker IP | 192.168.56.102 |
| Victim Host | DESKTOP-BA790JH (192.168.56.101) |
| Targeted Account | testuser |
| Failed Logons | 78+ |
| MITRE Techniques | T1110, T1059.001 |

## Evidence

### Hydra Attack from Kali
![Hydra Attack](https://github.com/user-attachments/assets/9fb278a5-a39c-4a02-a6d8-5a4ba6d6c018)

### Splunk — Failed Logon Events (Event ID 4625)
![4625 Events](https://github.com/user-attachments/assets/3b383941-635b-4f8c-af15-101ba6d5f1bd)

### Splunk — Process Creation Events (Event ID 4688)
![4688 Events](https://github.com/user-attachments/assets/c909bbcd-bc44-4ec5-b19b-af6a68914091)

### Brute Force Detection Alert
![Brute Force Alert](https://github.com/user-attachments/assets/2178d6c3-a5ad-4928-9f39-030625e7c41c)

### PowerShell Detection Alert
![PowerShell Alert](https://github.com/user-attachments/assets/b68f9a9f-7030-4e06-b738-b2dda18ff489)

## Containment Steps
- Isolate host from network
- Disable compromised account
- Block attacker IP at firewall
- Reset all local account passwords
- Review successful logons during attack window

## Lessons Learned
- Account lockout policy should be enforced (threshold: 3 attempts not 5)
- RDP should never be exposed without MFA
- Encoded PowerShell should be blocked via Constrained Language Mode
- Detection alerts should run in real-time not hourly for brute force
