# Q1 — Endpoint Detection and Response (EDR) & Endpoint Data

## 1. What is Endpoint Detection and Response (EDR)?

- EDR is basically a security tool that sits on endpoints like laptops, desktops and servers, and keeps watching everything that happens on them.
- It doesn't just block things — it records activity, notices suspicious behaviour, and gives the SOC team the ability to act on it quickly.
- The main idea behind EDR is that prevention alone is not enough anymore. Attackers often get past antivirus, so we need something that can catch them *after* they land and show us exactly what they did.
- Most EDR tools give you four things:
  - **Visibility** — a full record of processes, files, network connections, registry changes and user activity.
  - **Detection** — alerts based on behaviour, not just known bad files.
  - **Investigation** — process trees, timelines, and the ability to look back at what happened.
  - **Response** — actions like isolating a host, killing a process, or quarantining a file right from the console.

---

## 2. Why is endpoint data critical for a SOC analyst?

- Endpoints are where the actual damage happens — malware runs there, files get encrypted there, credentials get stolen there. So if we ignore endpoints, we miss most of the attack.
- Endpoint data gives **context** that network logs can't. For example, a network alert saying "host talked to a suspicious IP" is useful, but endpoint data can tell us *which process* made that connection and *which user* launched it.
- It helps with **root cause analysis**. With a proper process tree, an analyst can trace an attack from the first phishing email → Word macro → PowerShell → C2 connection, step by step.
- It supports **threat hunting**. Analysts can search across all endpoints for a specific behaviour (like "rundll32 spawning from Outlook") without waiting for an alert.
- It's a **core log source for the SIEM**. Sysmon, auditd and EDR agents all feed the SIEM, and correlation between endpoint and network logs is what makes good detections possible.
- It also helps with **compliance and forensics** — having retained telemetry means we can go back and answer "what happened on this machine last Tuesday at 3 AM?"

---

## 3. EDR vs Traditional Antivirus

- The easiest way to think about it: **antivirus is a lock on the door, EDR is a CCTV camera plus a security guard.**
- Antivirus waits for a known bad file and blocks it. EDR watches behaviour and lets the analyst decide.
- Here's a simple comparison:

|           Feature                                    |               Traditional Antivirus                       |                    EDR                                     |
|------------------------------------------------------|-----------------------------------------------------------|------------------------------------------------------------|
| **How it detects**                                   | Mostly signatures (known malware hashes and patterns)     | Behaviour, heuristics, ML, plus signatures                 |
| **Main goal**                                        | Prevent known threats from running                        | Detect, investigate and respond to threats                 |
| **Visibility**                                       | Very limited — basically "clean or infected"              | Deep telemetry: processes, files, network, registry, users |
| **Unknown / zero-day threats**                       | Usually misses them                                       | Can catch suspicious behaviour even if the file is new     |
| **Fileless attacks (e.g. PowerShell, LOLBins)**      | Weak                                                      | Strong — sees the process behaviour                        |
| **Response options**                                 | Quarantine file, block hash                               | Isolate host, kill process, rollback, remote response      |
| **Investigation support**                            | Almost none                                               | Full timeline, process tree, root cause                    |
| **Threat hunting**                                   | Not really a feature                                      | Built-in hunting and queries                               |
| **Analyst effort needed**                            | Low — mostly automated                                    | Higher — needs a SOC analyst to triage and respond         |
| **Typical role in a SOC**                            | Baseline protection layer                                 | Main detection and response tool on endpoints              |

---

## Key Takeaways

- EDR is about **seeing, understanding and reacting** to what happens on endpoints — antivirus is mainly about **blocking** known bad files.
- Endpoint telemetry is the **richest evidence source** an analyst has during an incident.
- In a modern SOC, **AV, EDR, SIEM and network tools** work together — no single tool covers everything.
- Even though AV is old, it's still useful as a **baseline** — but it's definitely not enough on its own against modern attacks like fileless malware or living-off-the-land techniques.
