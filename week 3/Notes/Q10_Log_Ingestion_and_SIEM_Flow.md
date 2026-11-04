# Q10 — Log Ingestion and the Flow from Log Source to SIEM Dashboard

## 1. What is Log Ingestion?

- **Log ingestion** is the process of **collecting logs from many different sources** (endpoints, servers, network devices, cloud services) and **bringing them into a central platform** — usually a SIEM (Security Information and Event Management).
- Think of it like this: every machine is writing its own diary. Log ingestion is how the SOC **gathers all those diaries into one library** where they can be searched, correlated, and alerted on.
- Without ingestion, a SOC analyst would have to SSH into 500 servers one by one to investigate an alert — impossible at scale.
- **Goals of log ingestion:**
  - **Centralize** — one place for all telemetry.
  - **Normalize** — put every source into a common schema (so Windows, Linux, firewall logs all look the same structurally).
  - **Enrich** — add context (GeoIP, asset criticality, threat intelligence, user identity).
  - **Retain** — keep data long enough for investigations and compliance.
  - **Enable detection** — feed correlation rules, dashboards, and alerts.

---

## 2. How Tools Like Wazuh and Splunk Collect Logs

- There are **two broad architectures** for log collection:
  - **Agent-based** — a small software agent installed on the endpoint forwards logs to the central platform.
  - **Agentless** — the central platform pulls logs directly (via syslog, API, or file share) without an agent.
- Most SOCs use a **mix of both** — agents on endpoints and servers, agentless for network devices and cloud APIs.

### Wazuh (open-source SIEM/XDR)

- **Wazuh agent** installed on endpoints (Windows, Linux, macOS) sends logs, file integrity monitoring (FIM) events, and security configuration assessment data to the **Wazuh manager**.
- **Agentless mode** collects from firewalls, routers, and cloud services via syslog, SSH, and APIs.
- The **Wazuh manager** decodes and rules-match the incoming data, generates alerts, and forwards them to:
  - The **Wazuh indexer** (OpenSearch-based) for storage.
  - The **Wazuh dashboard** (Kibana-based) for visualization.
- **Excellent for:** Sysmon, auditd, Windows Event Log, file integrity, vulnerabilities, and compliance checks. Free and popular in SOC labs.

### Splunk (commercial SIEM)

- **Splunk Universal Forwarder (UF)** is the agent installed on endpoints — it forwards logs to the **Splunk Indexer**.
- **Heavy Forwarders** are used when parsing/filtering is needed before sending.
- **Agentless collection** via **Splunk Connect for Syslog (SC4S)**, HTTP Event Collector (HEC), scripted inputs, and API-based modular inputs.
- The **Search Head** queries the indexers; **dashboards, alerts, and apps** (Enterprise Security) sit on top.
- **Excellent for:** high-volume enterprise data, deep search (SPL), rich apps (Enterprise Security, ITSI), and mature SOC workflows.

### Comparison table — Wazuh vs Splunk (log collection angle)

| Aspect | Wazuh | Splunk |
|--------|-------|--------|
| **Type** | Open-source SIEM/XDR | Commercial SIEM |
| **Agent** | Wazuh agent (lightweight) | Splunk Universal Forwarder (UF) |
| **Agentless** | Syslog, SSH, cloud APIs | SC4S, HEC, scripted inputs, modular inputs |
| **Storage** | OpenSearch indexer | Splunk Indexer |
| **Query language** | Wazuh rules + OpenSearch Query DSL | SPL (Search Processing Language) |
| **Best for** | SMB SOCs, labs, compliance, tight budgets | Enterprise SOCs, high volume, mature detection content |
| **Cost** | Free | Commercial (volume-based licensing) |

---

## 3. The Flow: From Log Source to SIEM Dashboard

- Here's the standard pipeline every SOC analyst should understand:
   Log Source → Collection → Transport → Parsing/Normalization → Enrichment → Indexing/Storage → Correlation & Alerting → Dashboard/Investigation


- **Step-by-step breakdown:**

| Stage | What happens | Example |
|-------|---------------|---------|
| **1. Log Source** | The system generates an event | Windows host logs Event ID 1 (Sysmon process create); Linux host writes to `audit.log`; firewall logs a dropped connection |
| **2. Collection** | Agent or agentless method picks up the log | Wazuh agent reads Sysmon channel; Splunk UF tails `/var/log/audit/audit.log`; firewall sends syslog |
| **3. Transport** | Logs are forwarded securely to the platform | Encrypted agent-to-manager connection; TLS syslog; HEC over HTTPS |
| **4. Parsing / Normalization** | Logs are decoded into a common schema | Sysmon EVTX → parsed fields (Image, CommandLine, User); auditd key=value → JSON |
| **5. Enrichment** | Context is added | GeoIP lookup on source IP; asset criticality; threat-intel IOC match; user → department mapping |
| **6. Indexing / Storage** | Data is stored for search | OpenSearch index (Wazuh) or Splunk index; retention policy applied |
| **7. Correlation & Alerting** | Detection rules run against the data | Wazuh rule: "5 failed SSH logins from same IP in 60s"; Splunk correlation search: "Sysmon 1 with parent winword.exe" |
| **8. Dashboard / Investigation** | Analyst sees alerts and pivots into data | Wazuh dashboard; Splunk ES incident review; drill-down into raw events |

- **What each layer must guarantee:**
  - **Collection/Transport:** reliability (no lost logs), integrity (TLS), and **time sync (NTP)** so correlation works.
  - **Parsing:** correct field extraction — otherwise rules can't fire.
  - **Enrichment:** context that turns "1.2.3.4" into "known malicious IP in Russia".
  - **Correlation:** detection logic — sigma rules, Wazuh rules, Splunk correlations.
  - **Dashboard:** what the analyst actually sees, plus the pivot path into the raw data.

---

## 4. Practical Considerations for a SOC Analyst

- **Coverage first, tuning second** — make sure all critical sources (Windows, Linux, firewall, DNS, cloud, identity) are flowing before writing fancy rules.
- **Time sync is non-negotiable** — misaligned clocks break correlation and timelines.
- **Normalize early** — using a common schema (e.g., **ECS — Elastic Common Schema**, **OCSF**, or Splunk CIM) makes rules portable and dashboards consistent.
- **Volume management** — not every log needs full retention. Firewall allowed-connections are high-volume; keep hot for a short time, cold for compliance.
- **Detection-as-code** — store correlation rules (Sigma, Wazuh XML, Splunk searches) in Git so the team can review and version them.
- **Dashboards should answer questions**, not just show graphs — "top failed logins today", "top talkers to new external IPs", "recent MITRE T1046 alerts".
- **Monitor the pipeline itself** — agent health, forwarder lag, indexer disk usage. If ingestion breaks, the SOC goes blind.

---

## Key Takeaways

- **Log ingestion is the SOC's lifeline** — it centralizes, normalizes, enriches, and stores telemetry from every source.
- **Wazuh** is agent-based (with agentless options), open-source, and ideal for endpoint and compliance visibility; **Splunk** is agent-based (UF) with rich commercial apps and SPL search.
- The flow is: **source → collection → transport → parse → enrich → index → correlate → dashboard**.
- **Coverage, time sync, normalization, and pipeline health** are the four things that make or break a SIEM.
- A SOC analyst's job at the dashboard is not just to read alerts — it's to **pivot into raw data**, confirm the alert, and feed tuning back into the pipeline.
