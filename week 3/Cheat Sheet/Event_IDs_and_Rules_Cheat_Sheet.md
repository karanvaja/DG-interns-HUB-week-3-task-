# SOC Analyst Cheat Sheet — Event IDs and Rules

This cheat sheet provides a quick reference for important Windows Event Logs, Sysmon events, Linux auditd records, and Snort alerts. It explains what each event means and when a SOC analyst should use it for threat hunting or incident response.

| Event ID / Rule | Source | What It Means | When to Use It |
|-----------------|--------|---------------|----------------|
| **4624** | Windows Security | Successful logon | Investigating normal vs. anomalous logins (e.g., logins at 3 AM or from unusual IPs). |
| **4625** | Windows Security | Failed logon | Detecting brute-force attempts or password spraying (tracking high volumes of 4625). |
| **4688** | Windows Security | Process creation | Tracking program execution (Note: Command Line must be enabled in Audit Policy). |
| **4720** | Windows Security | New user account created | Detecting persistence (MITRE T1136) and unauthorized local admin creation. |
| **1102** | Windows Security | Audit log cleared | Detecting anti-forensics. Attackers clear logs to hide their tracks; this is a high-severity alert. |
| **7045** | Windows System | New service installed | Detecting persistence. Malware often installs itself as a Windows service to survive reboots. |
| **1** | Sysmon | Process Create | Monitoring process execution with full command lines and parent-child relationships (detecting LOLBins). |
| **3** | Sysmon | Network Connection | Detecting C2 (Command & Control) beaconing or lateral movement by suspicious processes. |
| **11** | Sysmon | File Create | Detecting dropped files (e.g., malware payloads dropping .exe or .ps1 files in Temp folders). |
| **22** | Sysmon | DNS Query | Detecting DNS-based C2, DGA (Domain Generation Algorithms), or DNS tunneling. |
| **8** | Sysmon | CreateRemoteThread | Detecting process injection. One process starting a thread in another process (e.g., malware injecting into svchost). |
| **EXECVE** | Linux auditd | Command execution | Auditing Linux commands. Use `ausearch -m EXECVE` to see exactly what commands an attacker ran. |
| **USER_LOGIN** | Linux auditd | User login | Tracking successful Linux authentication and establishing an initial access timeline. |
| **USER_AUTH** | Linux auditd | Authentication attempt | Detecting failed logins and brute-force attempts on Linux (e.g., SSH failed passwords). |
| **1000001** | Snort | ICMP Ping Detected | Detecting network reconnaissance. While ping is common, a sudden spike from an unknown external IP can indicate a host discovery scan. |
| **1000002** | Snort | SSH Connection Attempt | Monitoring SSH access. Useful to detect brute-force attacks against Linux servers from the internet. |
| **1000003** | Snort | HTTP Traffic | Monitoring web traffic. Useful for detecting web shells, data exfiltration, or unusual HTTP requests. |
| **1000004** | Snort | Nmap SYN Scan Detected | Detecting port scanning (MITRE T1046). Alerts when an attacker is scanning the network for open ports. |

---

## Why This Cheat Sheet Matters
- **Speed:** During a live incident, analysts do not have time to Google every Event ID. This table acts as a quick lookup.
- **MITRE Mapping:** It helps map raw telemetry to known attacker behaviors (Tactics, Techniques, and Procedures).
- **Triage:** Knowing *when* to use an event helps analysts filter out background noise and focus on the actual threats.