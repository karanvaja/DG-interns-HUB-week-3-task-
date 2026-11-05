# LetsDefend — Incident Management

## Course Info
- **Platform:** LetsDefend
- **Course/Lesson:** Incident Management
- **Type:** Free Lesson
- **Date Completed:** 2026-11-05

## Objective
- Understand the lifecycle of a security incident.
- Learn the standard Incident Response (IR) frameworks (like NIST).
- Understand the roles of different teams (SOC, CSIRT, IT, PR) during an incident.
- Learn how to manage communication and documentation during a crisis.

---

## Key Concepts Learned

### 1. What is an Incident?
- A **Security Event** is any observable occurrence in a system or network (e.g., a user logging in, a firewall blocking an IP).
- A **Security Incident** is a violation or imminent threat of violation of computer security policies, acceptable use policies, or standard security practices.
- **Examples of Incidents:** Ransomware infection, data breach, DDoS attack, unauthorized access to a server.

### 2. The Incident Response Lifecycle (NIST SP 800-61)
The most widely accepted framework for Incident Management is the NIST lifecycle, which has 4 main phases:

1. **Preparation:**
   - Before an incident happens.
   - Creating policies, playbooks, and ensuring the right tools (SIEM, EDR) and people (SOC team) are in place. Conducting training and tabletop exercises.
2. **Detection and Analysis:**
   - Identifying that an incident is happening.
   - The SOC Analyst triages alerts, investigates the scope, and determines if it's a False Positive or a True Positive.
   - Gathering evidence and understanding the "who, what, when, where, and how".
3. **Containment, Eradication, and Recovery:**
   - **Containment:** Stopping the spread (e.g., isolating a computer from the network).
   - **Eradication:** Removing the threat (e.g., deleting malware, patching the vulnerability, resetting passwords).
   - **Recovery:** Restoring systems to normal operation and monitoring for reinfection.
4. **Post-Incident Activity (Lessons Learned):**
   - After the incident is resolved.
   - The team meets to discuss what went well, what failed, and how to improve. Updating policies, adding new SIEM rules, and training staff.

### 3. Roles and Responsibilities
Incident management is a team effort. Key roles include:
- **SOC Analyst (Tier 1/2):** First responders. They triage the alert, do the initial investigation, and escalate if necessary.
- **Incident Responder (Tier 3 / CSIRT):** Deep technical experts who lead the containment, eradication, and forensic analysis.
- **SOC Manager:** Oversees the team, handles resource allocation, and communicates with upper management.
- **IT / System Administrators:** Help apply patches, restore backups, and manage infrastructure.
- **Legal & PR Teams:** Handle compliance (e.g., GDPR notification) and public communication if it's a major data breach.
- **HR:** Involved if an insider threat is suspected.

### 4. Incident Severity & Prioritization
Not all incidents are equal. SOC teams use severity levels (e.g., P1, P2, P3, P4) to prioritize.
- **P1 (Critical):** Ransomware outbreak, active data exfiltration, compromise of Domain Controller. Requires immediate 24/7 response.
- **P2 (High):** Compromise of a single user endpoint with malware.
- **P3 (Medium):** Phishing email reported by a user (but not clicked).
- **P4 (Low):** Policy violations, minor scanning activity.

---

## Personal Notes / Reflections
- **Communication is Key:** I learned that during a major incident (like ransomware), the technical fix is only half the battle. Communicating clearly with management, legal, and other teams is just as important.
- **The "Lessons Learned" phase is crucial:** It's easy to fix the problem and move on. But taking the time to ask "How did this happen?" and "How can we prevent it next time?" is what makes a SOC mature.
- **Containment vs. Eradication:** It's important not to rush to delete malware (eradication) before understanding how it spread (containment). If you just delete the file without isolating the host, the attacker might just drop it again.