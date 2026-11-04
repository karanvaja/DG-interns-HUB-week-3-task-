# Q2 — Sysmon vs Standard Windows Event Logs

## 1. What is Sysmon?

- **Sysmon** (System Monitor) is a free tool from **Microsoft Sysinternals** that runs as a Windows service and driver on an endpoint.
- It logs **detailed system activity** to the Windows Event Log — things like process creation, network connections, file creation, registry changes, and DNS queries.
- Unlike normal Windows logging, Sysmon is **not enabled by default**. You install it manually and configure it with an **XML config file** to decide what to log and what to ignore.
- Sysmon is one of the **most valuable telemetry sources** for a SOC analyst on Windows, because it gives the kind of context that attackers can't easily hide — like the **full command line**, **parent process**, **user account**, and **file hash** of any process that runs.
- It's commonly paired with a **SIEM** (Splunk, Wazuh, Elastic) or **EDR-like pipelines** to power detection rules.

---

## 2. How does Sysmon differ from standard Windows Event Logs?

- Standard Windows Event Logs (Security, System, Application) are **built into Windows** and log what Microsoft decided to log — nothing more.
- Sysmon is an **add-on** that logs **much richer detail** about behaviour that normal logs miss or log poorly.
- Key differences:

|      Aspect           |                        Standard Windows Event Logs                             |                     Sysmon                                                        |                                                  
|-----------------------|--------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| **Availability**      | Built into Windows by default                                                  | Must be installed separately (Sysinternals)                                       |
| **Configurability**   | Limited — fixed event IDs                                                      | Fully configurable via XML (include/exclude rules)                                |
| **Process logging**   | Security 4688 gives basic process creation (command line is off by default)    | Event ID 1 gives full command line, parent process, hashes, user, integrity level |
| **Network logging**   | Almost nothing on outbound connections by default                              | Event ID 3 logs every network connection with source/dest IP, port, process       |
| **File logging**      | Very limited                                                                   | Event ID 11 logs file creation with path and process                              |
| **DNS logging**       | Not logged by default                                                          | Event ID 22 logs DNS queries with the requesting process                          |
| **Registry logging**  | Minimal                                                                        | Event ID 13 logs registry value set with process and value data                   |
| **Noise level**       | Can be noisy but shallow                                                       | Noisy unless tuned — that's why XML configs matter                                |
| **Best for**          | Compliance, auditing, baseline security events                                 | Threat detection, hunting, incident investigation                                 |

- Short version: **Windows Event Logs tell you *what happened*; Sysmon tells you *how it happened and by whom*** — which is what analysts actually need.

---

## 3. Sysmon Event IDs and what they monitor

- These are the most important Sysmon event IDs a SOC analyst should know:

| Event ID | Name               | What it monitors                                                                                      | Why it matters to a SOC                                            |
|----------|--------------------|-------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------|
| **1**    | Process Create     | Every new process launched — includes image path, command line, parent process, user, hashes          | Detects LOLBins, PowerShell abuse, suspicious parent-child chains  | 
| **3**    | Network Connection | Outbound/inbound TCP/UDP connections with source/dest IP, port, and the process making the connection | Detects C2 traffic, beaconing, unusual outbound connections        |
| **11**   | File Create        | Files created on disk, with full path and the process that created them                               | Detects dropped payloads, ransomware file writes, staging activity |
| **13**   | Registry Value Set | Registry values written or modified, with process and value data                                      | Detects persistence (Run keys), configuration tampering            | 
| **22**   | DNS Query          | DNS queries made by processes, with the queried domain and the process                                | Detects DGA domains, suspicious DNS lookups, malware resolution    |

- Additional useful IDs (worth knowing):

| Event ID  |       Name                    |        What it monitors                                            |
|-----------|-------------------------------|--------------------------------------------------------------------|
| 2         | File Creation Time Changed    | Timestomping attempts                                              |
| 5         | Process Terminated            | Process exits (useful for timeline analysis)                       |
| 7         | Image Loaded                  | DLL/module loads (detects DLL injection)                           |
| 8         | CreateRemoteThread            | Remote thread creation (classic injection indicator)               |
| 10        | Process Access                | One process opening another (credential dumping like LSASS access) |
| 12/14     | Registry Key Create/Delete    | Registry key creation or deletion                                  |
| 15        | FileCreateStreamHash          | Alternate data stream creation                                     |
| 17/18     | Pipe Created/Connected        | Named pipe activity (lateral movement, C2)                         |
| 23        | File Delete Archived          | File deletions (useful for ransomware tracking)                    |

---

## 4. Practical tips for using Sysmon in a SOC

- **Tuning is everything.** A default Sysmon install will flood the SIEM with noise. Use a community XML config (e.g., SwiftOnSecurity, Olaf Hartong) as a starting point and tune per environment.
- **Pair it with a SIEM.** Sysmon logs alone are hard to use; forwarding them to Splunk, Wazuh or Elastic makes them searchable and alertable.
- **Focus on behaviour, not just event IDs.** Event ID 1 with parent `winword.exe` spawning `powershell.exe` is far more interesting than the ID itself.
- **Watch the following hot events in detections:** 1 (suspicious command lines), 3 (unusual outbound), 8 and 10 (injection/credential access), 11 and 23 (file drops and deletions), 22 (beaconing domains).

---

## Key Takeaways

- Sysmon is **free, powerful, and configurable** — it turns Windows into a usable detection platform.
- It provides **context Windows logs were never designed to give** — parent processes, command lines, network connections, DNS.
- In a SOC, Sysmon is usually the **primary endpoint log source on Windows** until EDR is deployed.
- Event IDs **1, 3, 11, 13, 22** are the core five to master first.
