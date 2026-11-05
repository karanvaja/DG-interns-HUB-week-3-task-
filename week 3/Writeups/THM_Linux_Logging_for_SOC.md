# TryHackMe — Linux Logging for SOC

## Lab Info
- **Platform:** TryHackMe
- **Path:** SOC Level 1 > Linux Security Monitoring
- **Room Name:** Linux Logging for SOC
- **Difficulty:** Easy
- **Time to Complete:** ~45 minutes
- **Date Completed:** 2026-11-04

## Objective
- Explore the most common Linux log sources (authentication, runtime, and system logs).
- Learn how to use commands to filter and analyze logs directly on the host.
- Understand how tools like `auditd` monitor and report events.
- Learn why some logs are more useful to a SOC analyst than others.

---

## Task Writeups & Answers

### Task 1: Introduction
- **Overview:** This task introduces the room and explains how to start the lab machine. It highlights that Linux is widely used in servers and cloud environments, making Linux log investigation a critical skill for SOC analysts.
- **Answer:** No answer needed.

### Task 2: Working With Text Logs
- **Overview:** Unlike Windows, Linux logs are mostly stored as plain text files (usually in `/var/log/`). This task shows how to read and filter logs using commands like `cat`, `head`, and `grep`.
- **Questions & Answers:**
  - **Which time server domain did the VM contact to sync its time?**
    - **Answer:** `ntp.ubuntu.com`
  - **What is the kernel message from Yama in `/var/log/syslog`?**
    - **Answer:** `Becoming mindful.`

### Task 3: Authentication Logs
- **Overview:** The `/var/log/auth.log` file (or `/var/log/secure` on RHEL) is the most important log for SOC analysts. It records user logins, SSH sessions, sudo commands, and user management events.
- **Questions & Answers:**
  - **Which IP address failed to log in on multiple users via SSH?**
    - **Answer:** `10.14.94.82`
  - **Which user was created and added to the "sudo" group?**
    - **Answer:** `xerxes`

### Task 4: Common Linux Logs
- **Overview:** Explored other important logs like `kern.log` (kernel), `dpkg.log` (packages), and app-specific logs like web server access logs. It also covered Bash history (`~/.bash_history`), but noted it is often unreliable because attackers can bypass it (e.g., by adding a space before the command).
- **Questions & Answers:**
  - **According to the VM's package manager logs, which version of unzip was installed on the system?**
    - **Answer:** `6.0-28ubuntu4.1`
  - **What is the flag you see in one of the users' bash history?**
    - **Answer:** `THM{note to remember}`

### Task 5: Runtime Monitoring
- **Overview:** Standard logs are missing the "how" and "when" of process execution. To monitor runtime events (process creation, file changes, network connections), Linux uses System Calls (syscalls). The `execve` syscall is used to execute a program.
- **Questions & Answers:**
  - **Which Linux system call is commonly used to execute a program?**
    - **Answer:** `execve`
  - **Can a typical program open a file or create a process bypassing system calls? (Yes/Nay)**
    - **Answer:** `Nay`

### Task 6: Using Auditd
- **Overview:** `auditd` (Audit Daemon) is the built-in Linux solution for runtime monitoring. It uses rules configured in `/etc/audit/rules.d/` to monitor specific syscalls. Analysts use the `ausearch` command to read these logs easily.
- **Questions & Answers:**
  - **When was the `secret.thm` file opened for the first time? (MM/DD/YY HH:MM:SS)**
    - **Answer:** `08/13/25 18:36:54`
  - **What is the original file name downloaded from GitHub via wget?**
    - **Answer:** `naabu_2.3.5_linux_amd64.zip`
  - **Which network range was scanned using the downloaded tool?**
    - **Answer:** `192.168.50.0/24`

### Task 7: Conclusion
- **Overview:** Wrapped up the room. The key takeaway is that while Linux logging can be chaotic, the logs contain enough detail to detect threats if you know where to look.
- **Answer:** Complete the room!

---

## Key Takeaways
- **Top 3 log sources for SOC:** `auth.log` (authentication), app-specific logs (like web server logs), and runtime logs (`auditd`).
- **Bash History is unreliable:** Attackers can easily hide their tracks (e.g., using a leading space, modifying the shell, or running scripts). Do not rely on it for critical detection.
- **Syscalls are the truth:** Tools like `auditd`, Sysmon for Linux, and EDRs monitor system calls because attackers cannot bypass the kernel.
- **`ausearch` is your friend:** Use `ausearch -k <key>` to quickly filter through the noisy and verbose `auditd` logs.
- **Log Location:** Most Linux logs are kept in plain text in the `/var/log/` directory.

## Commands / Tools Used
```bash
# Filtering syslog
cat /var/log/syslog | grep CRON

# Searching for logins across all logs
grep -R -E "auth|login|session" /var/log

# Checking authentication logs
cat /var/log/auth.log | grep -E 'session opened|session closed'
cat /var/log/auth.log | grep "sshd"
cat /var/log/auth.log | grep -E '(passwd|useradd|usermod|userdel)'

# Using Auditd
ausearch -k proc_wget
ausearch -k file_sshconf
ausearch -i