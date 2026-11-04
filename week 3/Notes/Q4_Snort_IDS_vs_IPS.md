# Q4 — How Snort Works as an IDS/IPS + IDS vs IPS Mode

## 1. What is Snort?

- **Snort** is a free, open-source **intrusion detection and prevention system (IDS/IPS)** that inspects network traffic in real time.
- It was originally created by Martin Roesch and is now maintained by **Cisco (Talos)**.
- Snort works by **capturing packets, decoding protocols, and matching traffic against a rule set** — if a packet or a stream of packets matches a rule, Snort raises an alert (IDS mode) or blocks the traffic (IPS mode).
- It can also be used as a **packet sniffer** (like tcpdump) and a **packet logger** — but its main SOC role is as a detection engine.
- Modern Snort 3 supports **multi-threading**, Lua scripting, and better performance than the older Snort 2.x.
- In a SOC, Snort is usually deployed:
  - At the **network perimeter** (between firewall and internal network).
  - On **internal segments** (to catch lateral movement).
  - On **critical servers** (host-level IDS/IPS).

---

## 2. How does Snort work as an IDS/IPS?

- **Packet capture:** Snort grabs packets from a network interface (via `libpcap`/`DAQ` or inline drivers).
- **Decoding:** It decodes link-layer → IP → TCP/UDP/ICMP → application-layer protocols (HTTP, DNS, TLS, SMB, etc.).
- **Preprocessors:** Before rules run, preprocessors normalize traffic and handle things like IP defragmentation, TCP stream reassembly, port scan detection, and protocol anomalies.
- **Rule matching:** Snort runs each packet/stream through its **rule engine**. Rules are grouped by protocol and inspected in order. If a rule matches, an action is triggered.
- **Actions:**
  - `alert` → generate an alert (IDS)
  - `log` → log the packet
  - `pass` → ignore traffic
  - `drop` / `reject` → block traffic (IPS)
  - `activate` / `dynamic` → complex rule chaining
- **Output:** Alerts are written to a log file, syslog, database, or forwarded to a **SIEM** (Splunk, Wazuh, Elastic) in unified2/JSON format.

---

## 3. Difference between Snort in IDS mode vs IPS mode

- **IDS mode (passive):** Snort gets a **copy** of traffic (via a SPAN/mirror port or TAP), analyzes it, and raises alerts. It **cannot block anything** — it only sees and warns.
- **IPS mode (inline):** Snort sits **directly in the traffic path** (between firewall and switch). It analyzes every packet in real time and can **drop or reset** malicious traffic before it reaches the target.
- **Key differences:**

| Aspect | IDS Mode (Passive) | IPS Mode (Inline) |
|--------|---------------------|--------------------|
| **Placement** | Off to the side (SPAN port, TAP) | Directly in the traffic path |
| **Traffic handling** | Gets a copy of packets | Sees and forwards every packet |
| **Action capability** | Alert only — cannot block | Can alert, drop, reject, reset |
| **Latency impact** | None (traffic doesn't pass through it) | Adds latency (traffic must pass through) |
| **Risk of outage** | None — if it fails, network is fine | High — if it fails, traffic may stop |
| **False positive impact** | Just a noisy alert | **Could block legitimate traffic** |
| **Deployment complexity** | Easy | More complex (requires bypass/HA setup) |
| **Best for** | Monitoring, forensics, threat hunting | Active defense, blocking known attacks |
| **Example use case** | SOC watching internal traffic | Blocking exploit attempts at the perimeter |

- **Rule action difference:**
  - IDS mode ma `drop` rule no effect nathi — Snort sirf alert kare che.
  - IPS mode ma `drop` rule **actually traffic block kare che**.

---

## 4. When would a SOC use each?

- **Use IDS mode when:**
  - You want **visibility first** — during initial deployment, tuning, or when you don't trust the rule set yet.
  - The **network is sensitive** (production, OT, healthcare) and you can't risk a false-positive outage.
  - You need **forensics and threat hunting** — capturing traffic for later analysis.
  - You're feeding alerts into a **SIEM** for correlation but responding manually.
  - You're doing **compliance monitoring** (e.g., PCI-DSS requires IDS at some places).

- **Use IPS mode when:**
  - You have a **tuned rule set** and confidence that false positives won't break business.
  - You need to **actively block** known attacks (e.g., exploit kits, known C2 IPs, SQLi attempts).
  - The network segment is **high-risk** (internet-facing DMZ, web servers).
  - You've set up **bypass / fail-open** so that if Snort fails, traffic still flows.
  - You have a **change-control process** and monitoring around rule updates.

- **In practice, SOCs use both:**        
  - **IPS at the perimeter** for active blocking of known bad traffic.
  - **IDS on internal segments** for visibility and hunting without risking outages.
  - Many teams start with IDS, tune for weeks/months, and then slowly move to IPS on low-risk segments first.

---

## Key Takeaways

- **Snort is a rule-based network IDS/IPS** — it inspects traffic, matches rules, and either alerts (IDS) or blocks (IPS).
- **IDS = passive, safe, alert-only. IPS = inline, active, can drop traffic.**
- Deploy **IDS for visibility first**, then **IPS once rules are tuned** and bypass is in place.
- In a SOC, Snort alerts feed the **SIEM**, where they're correlated with endpoint and auth logs for full context.
- The biggest IPS risk is **false positives blocking legitimate traffic** — always plan for **fail-open / bypass**.
