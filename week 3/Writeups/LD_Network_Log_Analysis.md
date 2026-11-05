# LetsDefend — Network Log Analysis

## Course Info
- **Platform:** LetsDefend
- **Course/Lesson:** Network Log Analysis
- **Type:** Free Lesson (SOC Analyst Learning Path)
- **Date Completed:** 2026-11-05

## Objective
- Understand different types of network logs (Firewalls, Proxies, Routers).
- Learn how to analyze network traffic to detect malicious activity.
- Understand key network protocols and how attackers abuse them (C2, Exfiltration, Scanning).

---

## Key Concepts Learned

### 1. Types of Network Logs
In a SOC, network logs are crucial because they show how internal hosts communicate with the outside world (and each other).

- **Firewall Logs:**
  - Capture allowed and blocked connections based on IP, port, and protocol.
  - **Key fields:** Source IP, Destination IP, Source Port, Destination Port, Action (Allow/Deny), Bytes sent/received.
  - **SOC Use:** Spotting port scans, unauthorized access attempts, and data exfiltration (large outbound transfers).

- **Web Proxy Logs:**
  - Capture HTTP/HTTPS traffic from internal users.
  - **Key fields:** URL, HTTP Method (GET/POST), Status Code (200, 404, 403), User-Agent, Category (e.g., Malware, Phishing, Social Media).
  - **SOC Use:** Detecting users downloading malicious files, connecting to known C2 domains, or visiting phishing sites.

- **DNS Logs:**
  - Capture DNS queries (translating domain names to IP addresses).
  - **Key fields:** Query Name (Domain), Query Type (A, TXT, MX), Source IP.
  - **SOC Use:** Detecting **DNS Tunneling** (data exfiltration over DNS), Domain Generation Algorithms (DGA), and connections to malicious domains (e.g., malware C2).

- **VPN / Authentication Logs:**
  - Show who is connecting to the network remotely.
  - **SOC Use:** Spotting impossible travel (login from India and US within 10 minutes) or brute-force attacks against VPN gateways.

### 2. Common Attacks Seen in Network Logs

- **Port Scanning (Reconnaissance):**
  - **What it looks like:** One source IP trying to connect to many different destination ports (e.g., 21, 22, 80, 443, 3389) on a target host in a short time.
  - **Log Indicator:** Many "Deny" or "Drop" actions in firewall logs from the same source.

- **Command and Control (C2) Beaconing:**
  - **What it looks like:** An infected internal machine reaching out to an attacker's server at regular intervals (e.g., every 60 seconds) to get instructions.
  - **Log Indicator:** Repeated connections to a single external IP or domain, often using non-standard ports, with consistent small amounts of data.

- **Data Exfiltration:**
  - **What it looks like:** Attackers stealing sensitive data from the network.
  - **Log Indicator:** A massive spike in outbound data (bytes sent) from an internal server to an external IP, often during off-hours.

- **Web Application Attacks:**
  - **What it looks like:** SQL Injection, XSS, or Directory Traversal attempts.
  - **Log Indicator:** In proxy logs, you'll see payloads in the URL (e.g., `?id=1' OR '1'='1` or `../../etc/passwd`).

### 3. Key Indicators of Compromise (IOCs) in Network Logs
- **Suspicious IPs:** IPs with bad reputation (checked via Threat Intel like VirusTotal or AbuseIPDB).
- **Suspicious Ports:** Traffic going to ports that are not normally used for standard services (e.g., SSH on port 4444).
- **User-Agents:** Automated tools like `sqlmap`, `nmap`, or `curl` showing up in web logs instead of normal browsers.
- **Unusual Volume:** Abnormally large DNS queries or HTTP POST requests (indicating data theft).

---

## Personal Notes / Reflections
- **Context is Everything:** A connection to an external IP is not always bad. You have to look at the port, the volume of data, and the process making the connection. If `svchost.exe` (a Windows system process) suddenly tries to connect to an IP in Russia on port 8080, that's a red flag.
- **DNS is often overlooked:** Attackers love DNS because firewalls rarely block it. Analyzing DNS logs for weird, long random strings (DGA) or high volumes of TXT records (tunneling) is a vital SOC skill.
- **Beaconing:** I learned that malware doesn't just send all data at once. It "beacons" out slowly to avoid detection. Spotting the pattern (e.g., exactly every 5 minutes) is key.