# Q3 — auditd on Linux: What It Is, How It Works, and auditd vs syslog

## 1. What is auditd on Linux?

- **auditd** (the Linux Audit Daemon) is a **built-in kernel-level auditing service** on Linux that records security-relevant events happening on the system.
- It's part of the **Linux Audit Framework**, which has three main parts working together:
  - **Kernel audit subsystem** — hooks into system calls and generates audit records.
  - **auditd daemon** — listens for those records and writes them to `/var/log/audit/audit.log`.
  - **auditctl / ausearch / aureport tools** — used to configure rules and query/report on the logs.
- Unlike regular application logging, auditd works **inside the kernel**, so it can see **system calls, file access, user authentication, permission changes, and process execution** — even when the process itself doesn't log anything.
- It's commonly used for **compliance (PCI-DSS, CIS, STIG)**, **forensics**, and **detecting suspicious behaviour** on Linux hosts.

---  


## 2. How does auditd work?

- **Rule-based monitoring:** You define audit rules (using `auditctl` or by placing them in `/etc/audit/rules.d/`) that tell the kernel what to watch.
- **Kernel hooks trigger:** When a monitored event happens (e.g., a file is opened, a syscall runs, a user logs in), the kernel generates an **audit record**.
- **auditd receives the record:** The daemon collects these records and writes them to **`/var/log/audit/audit.log`**.
- **Userspace logging also included:** auditd also captures **user-space events** via **PAM** (authentication), **systemd**, and **SELinux/AppArmor** hooks — not just syscalls.
- **Querying and reporting:** Analysts use:
  - `ausearch` — search audit logs by key, user, time, etc.
  - `aureport` — generate summary reports.
  - `auditctl -l` — list active rules.
- **Common rule types:**
  - **File watch rules** — watch files/dirs for read/write/execute/attribute changes.
  - **Syscall rules** — log specific syscalls (e.g., `execve`, `open`, `connect`).
  - **Control rules** — manage audit daemon behaviour (e.g., buffer size, failure mode).
- **Example of a rule:**
  ```bash
  # Watch /etc/passwd for writes and attribute changes, tag it as "passwd_changes"
  -w /etc/passwd -p wa -k passwd_changes


3. Difference between auditd and syslog
     * syslog is a general-purpose message logging system. Applications, services, and the kernel write human-readable messages to syslog (e.g., "service started", "login failed").
     * auditd is a security-focused audit system that records specific system-level events with structured fields, not just free-text messages.
     * They solve different problems: syslog answers "what did the program say?" — auditd answers "what did the system actually do?"

|       Aspect            |                      syslog                                         |                     auditd                                                               |
|-------------------------|---------------------------------------------------------------------|------------------------------------------------------------------------------------------|
| **Purpose**             | General message logging                                             | Security auditing of system events                                                       |
| **Where it runs**       | Userspace — applications/services send messages to syslogd/rsyslogd | Kernel + userspace daemon (auditd)                                                       |
| **Log source**          | Application and service messages                                    | Kernel audit subsystem (syscalls, file access, PAM, SELinux)                             |
| **Log format**          | Free-text (human-readable)                                          | Structured audit records with `type=`, `key=`, `uid=`, `pid=`, `comm=`, etc.             |
| **Configurability**     | Per-app level (severity, facility)                                  | Rule-based (auditctl rules, XML rules in `/etc/audit/rules.d/`)                          |
| **Typical log file**    | `/var/log/syslog` (Debian), `/var/log/messages` (RHEL)              | `/var/log/audit/audit.log`                                                               |
| **Granularity**         | Whatever the app chooses to log                                     | Very fine-grained — syscall and path level                                               |
| **Tamper resistance**   | Moderate                                                            | Stronger — kernel-level, can be made immutable                                           |
| **Best for**            | Operational troubleshooting, general monitoring                     | Compliance, forensics, security detection                                                |
| **Can be disabled?**    | Yes, easily                                                         | Yes, but tamper-evident with `-e 2` (immutable mode)                                     |
| **Example event**       | `sshd[1234]: Failed password for root from 10.0.0.5`                | `type=SYSCALL msg=audit(...): arch=c000003e syscall=59 success=no comm="ssh" key="auth"` |


4. What a SOC analyst should do with auditd
    * Deploy sensible rules early — watch /etc/passwd, /etc/shadow, /etc/sudoers, /root, and privilege-escalation related syscalls.
    * Forward audit.log to the SIEM — either directly (Wazuh has a dedicated auditd decoder) or via audisp-remote/rsyslog.
    * Use ausearch for investigations — much faster than grepping raw logs:
    *       ausearch -k passwd_changes -ts today
            ausearch -m EXECVE --start recent
    * Alert on hot keys — e.g., failed sudo attempts, writes to auth files, execution of nc, curl or wget by unusual users.
    * Combine with Sysmon/EDR — on Linux, auditd is often the closest equivalent to Sysmon telemetry.


  # Key Takeaways
  * auditd is the Linux kernel's built-in auditing system — deep, structured, and rule-based.
  * It works by hooking into syscalls and authentication events, generating records that auditd writes to audit.log.
  * syslog is for general app/service messages; auditd is for security-relevant system activity. 
  * For SOC work on Linux, auditd is the primary endpoint telemetry source — tune it, forward it, and hunt with it.
