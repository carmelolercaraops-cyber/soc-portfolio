# Security Operations Center (SOC) Analyst Portfolio

Welcome to my SOC Analyst Portfolio. This repository showcases real-world threat detection, log correlation, and incident analysis workflows built across multi-layered telemetry sources (NIDS, Systemd Journal/PAM, and Kernel Auditd).

## 📁 Repository Structure
```
.
├── case-studies/
│   └── INC-2026-001-SSH-Bruteforce.md   # Reconnaissance & SSH Brute-Force Incident Report
└── README.md
```

## 🛠️ Detection & Telemetry Stack
- **Network Level:** Suricata NIDS (Signature Detection & Traffic Inspection)
- **Host / Auth Level:** Systemd Journal & PAM (journalctl, sshd, unix_chkpwd)
- **Kernel / Process Level:** Linux Audit Framework (auditd, ausearch, auditctl)

## 📋 Case Studies Overview

| Incident ID | Incident Name | Primary Tactics / Techniques | Severity | Status |
| :--- | :--- | :--- | :--- | :--- |
| [INC-2026-001](./case-studies/INC-2026-001-SSH-Bruteforce.md) | Reconnaissance & SSH Brute-Force Activity | Network Service Discovery (T1046), Password Guessing (T1110.001) | Medium | Closed |

## 🎓 About Me
- **Role:** SOC Tier 1 Analyst / Security Engineer
- **Focus:** Threat Detection, Tri-Layer Log Correlation, Incident Response
