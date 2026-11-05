# LetsDefend — SIEM 101

## Course Info
- **Platform:** LetsDefend
- **Course/Lesson:** SIEM 101
- **Type:** Free Lesson
- **Date Completed:** 2026-11-05

## Objective
- Understand what a SIEM is and why it is the heart of a SOC.
- Learn the core components of a SIEM architecture.
- Understand how logs are collected, parsed, normalized, and correlated to generate alerts.

---

## Key Concepts Learned

### 1. What is a SIEM?
- **SIEM** stands for **Security Information and Event Management**.
- It is a centralized platform that collects logs and security events from various sources across an organization's IT infrastructure (servers, endpoints, firewalls, routers, etc.).
- **Main Goal:** To provide a single pane of glass for security analysts to detect, investigate, and respond to threats in real-time.
- **Popular SIEM Tools:** Splunk, IBM QRadar, Wazuh, Elastic (ELK Stack), ArcSight, and Microsoft Sentinel.

### 2. The Core Components of a SIEM
A SIEM is not just one piece of software; it's an architecture made of several parts working together:

- **Log Sources:** Where the data comes from (e.g., Windows Event Logs, Linux auth.log, Firewalls, IDS/IPS, EDR agents).
- **Log Collector / Agent:** Software installed on endpoints or network devices that forwards the logs to the SIEM (e.g., Splunk Universal Forwarder, Wazuh Agent). For network devices, it might use Syslog.
- **Data Aggregation & Parsing:** The SIEM receives raw log data and extracts the important fields (e.g., Source IP, Destination IP, Username, Timestamp). 
- **Normalization:** Converting different log formats into a standard, common format. For example, a Windows log and a Linux log might both be normalized so that the "Source IP" field is recognized the same way across the board.
- **Correlation Engine:** The "brain" of the SIEM. It applies rules to the normalized data. It looks for patterns (e.g., 5 failed logins followed by a successful login from a new location).
- **Alerting & Dashboards:** When a correlation rule is triggered, the SIEM generates an alert. Dashboards provide visual summaries of security events.

### 3. How Log Correlation Works
- **The Problem:** A single log entry might be harmless. For example, a user logging into a VPN from a new location.
- **The Solution:** The SIEM correlates multiple events. If the user logs into the VPN from Russia, and 2 minutes later there's a failed admin login attempt on an internal server, the SIEM correlates these two events and raises a high-severity alert.
- **Rules:** Correlation rules are written by SOC engineers (often in Splunk's SPL, Elastic's KQL, or Sigma rules).

### 4. SIEM vs. EDR (Endpoint Detection and Response)
- **SIEM:** Focuses on **aggregating logs from everywhere** (network + endpoint) and finding the "big picture" across the entire organization.
- **EDR:** Focuses deeply on **what is happening on a single endpoint** (process trees, memory, file changes) and provides active response (isolating a host).
- **How they work together:** The EDR detects malware on a laptop and logs it. The log is sent to the SIEM. The SIEM correlates that log with a firewall log showing a weird connection, giving the analyst full visibility.

---

## Personal Notes / Reflections
- **The power of normalization:** I learned that logs come in hundreds of different formats. If a SIEM didn't normalize them, it would be impossible to search across different systems (e.g., searching for "192.168.1.5" would fail if one log calls it "src_ip" and another calls it "source_address").
- **Tuning is necessary:** SIEMs are noisy. If you just turn on all rules, you get overwhelmed. The most important task for a SOC analyst is tuning the SIEM to reduce false positives.
- **Data Retention:** SIEMs store a massive amount of data. I learned that logs are usually kept in "Hot" storage (fast, expensive) for a few months, and moved to "Cold" storage (slow, cheap) for compliance (usually 1 year).