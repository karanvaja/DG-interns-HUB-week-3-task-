# Q9 — Key Linux Log Files a SOC Analyst Should Monitor

## 1. Why Linux Log Monitoring Matters in a SOC

- Linux runs a huge portion of the internet's infrastructure — **web servers, databases, DNS, cloud workloads, containers, CI/CD pipelines** — so it's a **prime target** for attackers.
- Attackers who compromise a Linux host often leave traces in **standard system logs** — but only if someone is **collecting and reviewing them**.
- A SOC analyst monitoring Linux needs to know **which log file tells which story** — auth for logins, syslog for general services, audit.log for kernel-level activity, kern.log for hardware and kernel events, and service-specific logs (Apache, MySQL, etc.) for application behaviour.
- **Without log forwarding to a SIEM**, these logs are only useful on the host itself. The SOC goal is to **centralize, correlate, and alert** on them.

---

## 2. Key Linux Log Files and What They Tell You

- Below is the minimum set every Linux-aware SOC analyst should know — but every environment adds more (nginx, mysql, docker, journald, etc.).
- **Locations vary slightly by distro:** `/var/log/auth.log` is Debian/Ubuntu; on RHEL/CentOS it's `/var/log/secure`. Similarly, `/var/log/syslog` is Debian/Ubuntu; RHEL uses `/var/log/messages`.

### Master Table

| Log File Path | Purpose | Common Events |
|---------------|---------|---------------|
| **`/var/log/auth.log`** (Debian/Ubuntu) / **`/var/log/secure`** (RHEL/CentOS) | Authentication and authorization events — who logged in, how, and whether it succeeded | SSH login success/failure, `sudo`/`su` usage, PAM errors, failed password attempts, invalid users, session open/close, cron auth |
| **`/var/log/syslog`** (Debian/Ubuntu) / **`/var/log/messages`** (RHEL/CentOS) | General system and service messages — the catch-all log | Service start/stop, cron jobs, network events, application messages, kernel warnings, mail, DHCP events |
| **`/var/log/audit/audit.log`** | Kernel-level auditing records from **auditd** — syscalls, file access, auth, SELinux | `execve` process executions, file open/write on watched paths (`/etc/passwd`, `/etc/sudoers`), `setuid` syscalls, SELinux denials, PAM events, audit rule changes |
| **`/var/log/kern.log`** | Kernel messages only — hardware and low-level system events | Kernel panics, OOM killer, USB insertion, driver load, firewall (iptables/nftables) logs, segmentation faults, hardware errors |
| **`/var/log/apache2/access.log`** (Debian) / `/var/log/httpd/access_log` (RHEL) | Web server access log — every HTTP request | Client IP, request method/path, status code, user-agent, bytes sent, suspicious paths (`/admin`, `/.env`), SQLi attempts, scanning patterns |

### Additional Log Files Worth Knowing

| Log File Path | Purpose | Common Events |
|---------------|---------|---------------|
| `/var/log/apache2/error.log` | Web server error log | PHP errors, 500 errors, blocked requests, module failures |
| `/var/log/nginx/access.log` + `error.log` | Same as above for nginx | Same |
| `/var/log/faillog` | Failed login attempts summary | Repeated failed logins per user |
| `/var/log/lastlog` | Last login info per user | First-time logins from unusual IPs |
| `/var/log/wtmp` / `/var/log/btmp` | Successful / failed login records (binary) | Audit trail of logins (`last`, `lastb`) |
| `/var/log/cron` | Cron job execution | Suspicious cron entries, persistence attempts |
| `/var/log/mail.log` | Mail server logs | Spam, relaying, exploit attempts |
| `/var/log/mysql/error.log` | MySQL errors | Auth failures, query errors |
| `journalctl` (systemd journal) | Modern unified logging | All services, kernel, boot logs |

---

## 3. What a SOC Analyst Looks For in Each Log

- **auth.log / secure:**
  - **Brute-force patterns** — many failed logins from one IP or to one user.
  - **Successful login from a new country or unusual IP** — possible compromised credentials.
  - **`sudo` to root by non-admin users**, or `sudo -l` enumeration.
  - **SSH login as `root` from external IPs** — red flag unless explicitly allowed.
  - **Invalid user attempts** — automated scanners.

- **syslog / messages:**
  - **Service crashes or restarts** — possible tampering.
  - **Cron job additions or unusual schedules** — persistence.
  - **Unexpected DHCP or network changes** — lateral movement indicators.
  - **Logging service stopped or restarted** — log evasion.

- **audit.log:**
  - **Execution of shells, `curl`, `wget`, `nc`, `python -c`, `base64 -d`** — common in post-exploitation.
  - **Writes to `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`, `/root/.ssh/`** — privilege escalation and persistence.
  - **`setuid`/`setgid` calls to unusual binaries** — privilege escalation.
  - **SELinux/AppArmor denials** — policy bypass attempts.
  - **Audit rule changes** — attacker trying to disable monitoring (`auditctl -D`).

- **kern.log:**
  - **Firewall drops/blocks** — incoming scanning and attack patterns.
  - **OOM killer** — possible memory exhaustion attack (DoS).
  - **USB insertion events** — possible exfiltration or malware delivery.
  - **Segfaults in services** — exploit attempts or instability.

- **Apache/nginx access.log:**
  - **Sequential 404s** — directory scanning.
  - **SQLi / XSS payloads in query strings** (`UNION SELECT`, `<script>`).
  - **Requests to sensitive paths** (`/.env`, `/.git/`, `/admin`, `/phpmyadmin`).
  - **Unusual User-Agents** (`sqlmap`, `nikto`, `curl`).
  - **High 4xx/5xx rate from one IP** — fuzzing or exploitation attempts.

---

## 4. How a SOC Operationalizes Linux Logs

- **Forward everything to the SIEM** — Wazuh, Splunk UF, rsyslog, or Fluent Bit. Local logs alone are useless for a SOC.
- **Use the right tool for the right log:**
  - **`journalctl`** for systemd-managed services.
  - **`ausearch` / `aureport`** for auditd.
  - **`grep`, `awk`, `goaccess`** for quick manual triage.
- **Set detections for the "big five":**
  - SSH brute force (auth.log)
  - Root or admin login from external IP (auth.log)
  - `sudo` to root by non-admin (auth.log + audit.log)
  - Writes to auth files or SSH keys (audit.log)
  - Web scanning / exploitation attempts (apache/nginx access.log)
- **Enable log rotation awareness** — attackers sometimes wait for logs to rotate; the SIEM copy is the durable record.
- **Watch for log tampering** — `auditd` can be made immutable (`-e 2`), and any unusual `auditctl` change should alert.
- **Watch the clock** — ensure NTP is running on all hosts; misaligned time breaks correlation.

---

## Key Takeaways

- The **five core Linux logs** every SOC analyst must know: **auth.log, syslog, audit.log, kern.log, and web access.log**.
- **auth.log = logins and sudo.** **syslog = general services.** **audit.log = kernel-level activity.** **kern.log = hardware/kernel.** **access.log = web traffic.**
- Every log is only as useful as the **SIEM pipeline** feeding it — **forward, parse, correlate, alert**.
- **auditd is the Linux equivalent of Sysmon** — deploy sensible rules and forward `audit.log` as a priority.
- **Attackers often try to disable or rotate logs** — immutable audit rules and central log collection are your defenses.
