# LetsDefend — Incident Response on Linux

## Course Info
- **Platform:** LetsDefend
- **Course/Lesson:** Incident Response on Linux
- **Type:** Free Lesson
- **Date Completed:** 2026-11-05

## Objective
- Learn the step-by-step process of investigating a compromised Linux system.
- Understand how to identify malicious processes, network connections, and persistence mechanisms.
- Master essential Linux commands used for live forensics and Incident Response (IR).

---

## Key Concepts Learned

### 1. The Initial Triage (Live Response)
When a Linux server is suspected to be compromised, the first goal is to understand the current state of the system without shutting it down (which destroys volatile memory data).
- **Who is logged in?** 
  - Command: `w` or `who` (Shows currently logged-in users and what they are doing).
- **What processes are running?**
  - Command: `ps aux` or `top` (Look for unusual process names, high CPU usage, or processes running as `root` that shouldn't be).
- **What network connections are active?**
  - Command: `netstat -tulpn` or `ss -tulpn` (Look for listening ports, especially ones communicating with external IPs or on unusual ports).
  - Command: `lsof -i` (Lists open files and the processes making network connections).

### 2. Finding Malicious Activity (Processes & Files)
Attackers often rename malicious binaries to look like legitimate system processes (e.g., naming malware `kworker` or `sshd`).
- **Check Process Paths:** Use `ps -ef` to see the full path. If `sshd` is running from `/tmp/sshd` instead of `/usr/sbin/sshd`, it's likely malware.
- **Check Hidden Files:** Attackers hide files using a dot (`.`) prefix or in hidden directories.
  - Command: `ls -la` (List all files, including hidden ones).
- **Find recently modified files:** Attackers drop tools or modify configs.
  - Command: `find / -mtime -1` (Finds files modified in the last 24 hours).
- **Check Deleted Binaries:** Sometimes attackers delete the executable while it's still running (to hide it). You can check this in `/proc/[PID]/exe`.

### 3. Identifying Persistence Mechanisms
Attackers want their access to survive a reboot. Where do they hide?
- **Cron Jobs:** Very common for persistence.
  - Command: `crontab -l` (Check current user's cron), `cat /etc/crontab`, `ls -la /etc/cron.*`
- **Systemd Services:** 
  - Command: `systemctl list-units --type=service` (Look for unusual or newly created services).
  - Check `/etc/systemd/system/` for malicious `.service` files.
- **SSH Authorized Keys:** Attackers add their public key to log in without a password.
  - Command: `cat ~/.ssh/authorized_keys` (Check for unknown keys).
- **Bash Profiles:** Modifying `.bashrc` or `.profile` to run malware every time a user logs in.
- **Init/RC Scripts:** Modifying `/etc/init.d/` or `/etc/rc.local`.

### 4. Log Analysis (Where to look)
- **Authentication Logs (`/var/log/auth.log` or `/var/log/secure`):** Look for successful SSH logins from unusual IPs or brute-force attempts.
- **System Logs (`/var/log/syslog` or `/var/log/messages`):** Look for service crashes or restarts.
- **Auditd Logs (`/var/log/audit/audit.log`):** If `auditd` is enabled, use `ausearch -m EXECVE` to see what commands were run.
- **Command History:** Check `~/.bash_history` for the user, but remember attackers often clear this.
- **Web Server Logs:** If it's a web server, check `/var/log/apache2/` or `/var/log/nginx/` for web shell access or SQL injection attempts.

### 5. Containment and Eradication
- **Containment:** If it's a critical server, you might need to isolate it by blocking its IP at the firewall or disabling its network interface (`ip link set eth0 down`).
- **Eradication:** Kill the malicious process (`kill -9 <PID>`), remove the malicious files, remove the attacker's SSH keys, and remove the malicious cron jobs.
- **Rootkit check:** Use tools like `chkrootkit` or `rkhunter` to scan for common rootkits, though a full rebuild is often the safest option after a root compromise.

---

## Personal Notes / Reflections
- **Volatile Data is Key:** I learned that if you reboot a compromised Linux machine, you lose all the processes, active network connections, and memory-resident malware. Live response must be done carefully first.
- **Look for the anomalies:** Normal Linux servers are boring. `top` usually shows the same processes. If you see a process consuming 99% CPU named `xmrig` or `kdevtmpfsi`, that's a crypto-miner.
- **The "Hidden in Plain Sight" trick:** Attackers love hiding binaries in `/tmp`, `/dev/shm`, or `/var/tmp` because these directories are often writable by everyone and sometimes mounted with execution permissions.
- **Trust nothing:** A compromised server might have a modified `ps` or `ls` command (a rootkit). Advanced IR analysts use statically compiled binaries (like BusyBox) from a USB drive to ensure they are seeing the real truth.