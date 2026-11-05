# TryHackMe — Intro to Log Analysis

## Lab Info
- **Platform:** TryHackMe
- **Path:** SOC Level 1 > Log Analysis
- **Room Name:** Intro to Log Analysis
- **Difficulty:** Easy
- **Time to Complete:** ~60 minutes
- **Date Completed:** 2026-11-05

## Objective
- Learn log analysis best practices.
- Discover the essential tools for log analysis.
- Gain hands-on experience in analyzing logs by using multiple tools and technologies.

---

## Task Writeups & Answers

### Task 1: Introduction
- **Overview:** Introduction to log analysis, which involves collecting, parsing, and processing log files to monitor metrics and identify security incidents.
- **Answer:** No answer needed.

### Task 2: Log Analysis Basics
- **Overview:** Covered the definition of logs, why they are important (troubleshooting, security, threat hunting, compliance), and different types of logs (Application, Audit, Security, Server, System, Network, Database, Web Server).
- **Answer:** No answer needed.

### Task 3: Investigation Theory
- **Overview:** Discussed creating timelines, handling timestamps (normalizing to UTC/UNIX), the concept of **Super Timelines** (using tools like Plaso), data visualization (dashboards), log monitoring/alerting, and using threat intelligence (IOCs like IPs, hashes).
- **Questions & Answers:**
  - **What's the term for a consolidated chronological view of logged events from diverse sources, often used in log analysis and digital forensics?**
    - **Answer:** `Super Timeline`
  - **Which threat intelligence indicator would `5bf1f93c09ad14065c0491b7649d4933` and `763f8bdbc98d105a8e82f16157e9bbbe` be classified as?**
    - **Answer:** `File Hashes`

### Task 4: Detection Engineering
- **Overview:** Covered common log file locations for various services (Nginx, Apache, MySQL, Linux, etc.). We explored abnormal user behavior (failed logins, unusual times, geographic anomalies) and common attack signatures in logs like SQL Injection, XSS, and Path Traversal.
- **Questions & Answers:**
  - **What is the default file path to view logs regarding HTTP requests on an Nginx server?**
    - **Answer:** `/var/log/nginx/access.log`
  - **A log entry containing `%2E%2E%2F%2E%2E%2Fproc%2Fself%2Fenviron` was identified. What kind of attack might this infer?**
    - **Answer:** `Path Traversal`

### Task 5: Automated vs. Manual Analysis
- **Overview:** Compared the pros and cons of using automated tools (AI/ML, fast, but can be expensive and have false positives) versus manual analysis (requires more time and expertise, but offers deeper contextual understanding).
- **Questions & Answers:**
  - **A log file is processed by a tool which returns an output. What form of analysis is this?**
    - **Answer:** `Automated`
  - **An analyst opens a log file and searches for events. What form of analysis is this?**
    - **Answer:** `Manual`

### Task 6: Log Analysis Tools: Command Line
- **Overview:** Practiced using essential Linux commands for log analysis: `cat`, `less`, `tail`, `wc`, `cut`, `sort`, `uniq`, `sed`, `awk`, and `grep`. We used these to filter, extract, and transform log data.
- **Questions & Answers:**
  - **Use `cut` on the `apache.log` file to return only the URLs. What is the flag that is returned in one of the unique entries?**
    - **Answer:** `c701d43cc5a3acb9b5b04db7f1be94f6`
  - **In the `apache.log` file, how many total HTTP 200 responses were logged?**
    - **Answer:** `52`
  - **In the `apache.log` file, which IP address generated the most traffic?**
    - **Answer:** `145.76.33.201`
  - **What is the complete timestamp of the entry where `110.122.65.76` accessed `/login.php`?**
    - **Answer:** `31/Jul/2023:12:34:40 +0000`

### Task 7: Log Analysis Tools: Regular Expressions
- **Overview:** Introduced Regular Expressions (Regex) for pattern matching in log files. Demonstrated how to use `grep -E` for regex searching and how regex is used in parsing tools like Grok.
- **Questions & Answers:**
  - **How would you modify the original `grep` pattern above to match blog posts with an ID between 20-29?**
    - **Answer:** `post=2[0-9]`
  - **What is the name of the filter plugin used in Logstash to parse unstructured log data?**
    - **Answer:** `Grok`

### Task 8: Log Analysis Tools: CyberChef
- **Overview:** Explored CyberChef, a web-based tool for analyzing data. We used it to decode Base64 strings and to apply Regex to extract specific data (like IP addresses) from large log files using the "Extract IP addresses" operation.
- **Questions & Answers:**
  - **Locate the "loganalysis.zip" file under `/root/Rooms/introloganalysis/task8` and extract the contents.**
    - **Answer:** `No answer needed`
  - **Upload the log file named "access.log" to CyberChef. Use regex to list all of the IP addresses. What is the full IP address beginning in 212?**
    - **Answer:** `212.14.17.145`
  - **Using the same log file from Question #2, a request was made that is encoded in base64. What is the decoded value?**
    - **Answer:** `THM{CYBERCHEF_WIZARD}`
  - **Using CyberChef, decode the file named "encodedflag.txt" and use regex to extract by MAC address. What is the extracted value?**
    - **Answer:** `08-2E-9A-4B-7F-61`

### Task 9: Log Analysis Tools: Yara and Sigma
- **Overview:** Introduced YARA and Sigma. Sigma uses YAML to create generic SIEM detection rules. YARA uses pattern matching (Regex/Strings) to identify malware or specific log patterns.
- **Questions & Answers:**
  - **What languages does Sigma use?**
    - **Answer:** `YAML`
  - **What keyword is used to denote the "title" of a Sigma rule?**
    - **Answer:** `title`
  - **What keyword is used to denote the "name" of a rule in YARA?**
    - **Answer:** `rule`

### Task 10: Conclusion
- **Overview:** Summary of the log analysis methodology, tools, and importance of data collection.
- **Answer:** No answer needed.

---

## Key Takeaways
- **The Power of the Command Line:** Tools like `grep`, `awk`, `sed`, and `sort` are incredibly powerful for quick, on-the-fly log analysis without needing a SIEM.
- **Regex is Essential:** Regular expressions are a must-have skill for extracting specific patterns (like IP addresses) from unstructured text.
- **Sigma and YARA:** Sigma is great for writing generic SIEM rules (YAML), while YARA is excellent for pattern matching and malware identification.
- **Threat Intel:** IOCs (IPs, Hashes, Domains) help narrow down the noise when analyzing logs.
- **Super Timelines:** Combining logs from multiple sources chronologically creates a "Super Timeline," which is crucial for complex incident response.

## Commands / Tools Used
```bash
# Basic File Viewing
cat apache.log
less apache.log
tail -f -n 5 apache.log
wc apache.log

# Extracting and Sorting Data
cut -d ' ' -f 1 apache.log
cut -d ' ' -f 1 apache.log | sort -n -r
cut -d ' ' -f 1 apache.log | sort -n -r | uniq -c

# Searching with Grep and Regex
grep "admin" apache.log
grep -c "admin" apache.log
grep -n "admin" apache.log
grep -v "/index.php" apache.log
grep -E 'post=1[0-9]' apache-ex2.log