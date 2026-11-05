# LetsDefend — Security Solutions

## Course Info
- **Platform:** LetsDefend
- **Course/Lesson:** Security Solutions
- **Type:** Free Lesson
- **Date Completed:** 2026-11-05

## Objective
- Understand the different types of security solutions used in a modern SOC.
- Learn how Firewalls, IDS/IPS, EDR, SIEM, SOAR, and Sandboxing work together.
- Understand the difference between preventive, detective, and responsive controls.

---

## Key Concepts Learned

### 1. The Big Picture of Security Solutions
In a SOC, no single tool can stop all attacks. Security is built in layers (Defense in Depth). A typical enterprise uses multiple solutions to **Prevent**, **Detect**, and **Respond** to threats.

### 2. Core Security Solutions Explained

- **Firewall:**
  - **What it does:** Acts as a barrier between the internal network and the internet. It filters traffic based on IP addresses, ports, and protocols (Layer 3/4).
  - **SOC Use:** Blocks known malicious IPs, restricts unauthorized access to internal servers, and segments the network.
  - **Types:** Next-Generation Firewall (NGFW) can also inspect application-layer traffic (Layer 7).

- **IDS / IPS (Intrusion Detection & Prevention Systems):**
  - **IDS (Detection):** Passive. It monitors a copy of the traffic and alerts the SOC if it sees malicious activity (like a port scan or SQL injection).
  - **IPS (Prevention):** Inline. It sits directly in the traffic path and actively drops or blocks malicious packets in real-time.
  - **SOC Use:** Detecting known attack signatures (Signature-based) or unusual behavior (Anomaly-based).

- **EDR (Endpoint Detection and Response):**
  - **What it does:** Installed as an agent on endpoints (laptops, servers). It monitors processes, file changes, and network connections at a very deep level.
  - **SOC Use:** Threat hunting, detecting fileless malware, and **isolating** a compromised host directly from the console.
  - **Sysmon:** A free Microsoft tool that provides similar (but non-responsive) telemetry for Windows endpoints.

- **SIEM (Security Information and Event Management):**
  - **What it does:** The "brain" of the SOC. It collects logs from *everything* (Firewall, EDR, Servers, Active Directory) and correlates them.
  - **SOC Use:** Generating alerts. For example, correlating a "Failed Login" log from a server with a "Firewall Block" log from the same IP.
  - **Examples:** Splunk, Wazuh, Elastic, IBM QRadar.

- **SOAR (Security Orchestration, Automation, and Response):**
  - **What it does:** Automates repetitive tasks. If a SIEM generates an alert, SOAR can automatically block the IP on the firewall, disable the user account, and send an email to the analyst.
  - **SOC Use:** Reduces "Alert Fatigue" and speeds up response time (Playbooks).

- **Sandboxing:**
  - **What it does:** A safe, isolated virtual environment. If a user downloads a suspicious file, it is executed in the Sandbox first.
  - **SOC Use:** Analyzing zero-day malware. The SOC observes what the file tries to do (e.g., connect to a C2 server) without risking the actual network.

### 3. HoneyPots
- **What it is:** A decoy system designed to look like a real target (e.g., a fake database server).
- **SOC Use:** Since no legitimate user should ever access a honeypot, any traffic going to it is highly suspicious. It acts as an early warning system for internal scanning or lateral movement.

---

## Personal Notes / Reflections
- **Prevention vs Detection:** I learned that while a Firewall and IPS are great for *preventing* known attacks, EDR and SIEM are crucial for *detecting* attacks that bypass the perimeter (like a phishing email leading to a malicious script).
- **The role of SOAR:** SOAR is a lifesaver for SOC teams. Manually checking 1000 alerts a day is impossible. Automating the basic checks (like IP reputation lookup) makes the job much easier.
- **Layering:** A hacker might bypass the firewall using an encrypted connection, but the EDR might catch the malicious process running on the endpoint, and the SIEM will tie it all together.