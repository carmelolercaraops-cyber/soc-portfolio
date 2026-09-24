# INCIDENT REPORT: INC-2026-001 - Reconnaissance & SSH Brute-Force Activity

## 1. Incident Overview & Metadata
| Parameter | Details |
| :--- | :--- |
| **Incident ID** | INC-2026-001 |
| **Severity** | Medium |
| **Status** | Closed / Escalated to Tier 2 |
| **Event Date/Time** | 2026-09-24 20:06:45 - 20:07:00 CEST |
| **Detection Engines** | Suricata NIDS, Linux Systemd Journal/PAM, Auditd Kernel Monitor |
| **Assigned Analyst** | SOC Tier 1 Analyst |

---

## 2. Executive Summary
On September 24, 2026, at 20:06:45 CEST, network and host security sensors detected a coordinated reconnaissance and brute-force attempt originating from loopback interface 127.0.0.1. 

The activity initiated with a TCP port scan targeting ports 1-1000, followed immediately at 20:06:53 CEST by an SSH credential-stuffing attack against user account th3g3ntl3man. Host-level kernel monitoring (auditd) captured process execution lineages for both nmap (PID 355229) and hydra (PID 355309). No successful authentication occurred (res=failed), and the SSH daemon automatically enforced source penalization (srclimit_penalise).

---

## 3. Tri-Layer Telemetry Extraction & Analysis

### 3.1 Network Telemetry (Suricata IDS)
Extracted alert signatures from /var/log/suricata/fast.log:

09/24/2026-15:09:37.775191 [**] [1:1000002:1] SOC LAB - Nmap Scan Detected [**] {TCP} 127.0.0.1:50545 -> 127.0.0.1:350
09/24/2026-15:10:06.155283 [**] [1:1000001:1] SOC LAB - SSH Brute Force Attempt [**] {TCP} 127.0.0.1:38776 -> 127.0.0.1:22

### 3.2 Host Authentication Telemetry (journalctl / PAM)
Authentication log breakdown extracted from Systemd journal:

Sep 24 20:06:55 kali unix_chkpwd[355334]: password check failed for user (th3g3ntl3man)
Sep 24 20:06:57 kali sshd-session[355327]: Failed password for th3g3ntl3man from 127.0.0.1 port 48958 ssh2
Sep 24 20:07:00 kali sshd[84649]: srclimit_penalise: 127.0.0.1/32: activating ipv4 penalty of 18.071 seconds for penalty: failed authentication

### 3.3 Kernel Process Execution Telemetry (auditd)
Execution events extracted via ausearch confirming precise binary execution and process lineage:

time->Thu Sep 24 20:06:45 2026
type=EXECVE msg=audit(1790273205.112:112471): argc=7 a0="/usr/bin/env" a1="sh" a2="/usr/bin/nmap" a3="-sS" a4="-p" a5="1-1000" a6="127.0.0.1"
type=SYSCALL msg=audit(...): success=yes pid=355229 comm="nmap" exe="/usr/bin/env" key="exec_monitor"

time->Thu Sep 24 20:06:53 2026
type=EXECVE msg=audit(1790273213.483:112472): argc=9 a0="hydra" a1="-l" a2="th3g3ntl3man" a3="-P" a4="/tmp/passwords.txt" a5="127.0.0.1" a6="ssh" a7="-t" a8="4"
type=SYSCALL msg=audit(...): success=yes pid=355309 comm="hydra" exe="/usr/bin/hydra" key="exec_monitor"

---

## 4. Indicators of Compromise (IoCs)
* Source IP: 127.0.0.1 (Localhost loopback simulation)
* Target User Account: th3g3ntl3man
* Target Port: 22/TCP (SSH)
* Executed Binaries: /usr/bin/nmap (PID 355229), /usr/bin/hydra (PID 355309)
* Artifact File: /tmp/passwords.txt

---

## 5. MITRE ATT&CK Mapping
* Reconnaissance (TA0043)
  * Network Service Discovery (T1046)
* Credential Access (TA0006)
  * Brute Force: Password Guessing (T1110.001)

---

## 6. Containment, Mitigation & Recommendations
1. Validation: Confirmed 100% failure rate across password-guessing attempts (res=failed).
2. Mitigation: OpenSSH srclimit_penalise successfully applied active dynamic connection dropping.
3. Recommendations:
   * Enforce SSH Key Authentication (PubkeyAuthentication yes) and disable plain password login in /etc/ssh/sshd_config.
   * Deploy Fail2ban or active firewall rules to persistently block IPs after multiple failed authentication attempts.
   * Restrict access to internal enumeration binaries for unprivileged users where applicable.
