# Q7 — MITRE ATT&CK T1046 (Network Service Discovery) and Snort Rule Mapping

## 1. What is MITRE ATT&CK T1046 (Network Service Discovery)?

- **MITRE ATT&CK** is a publicly maintained knowledge base of **real-world adversary tactics, techniques, and procedures (TTPs)**. It's organized into **tactics** (the "why") and **techniques** (the "how").
- **T1046** is the technique **"Network Service Discovery"** under the tactic **Discovery (TA0007)**.
- **What it means:** The adversary is **probing a network to find services that are running** — open ports, live hosts, listening daemons (SSH, RDP, SMB, HTTP, databases, etc.).
- **Why attackers do it:** To **map the environment** before moving to the next phase — exploitation, lateral movement, or privilege escalation.
- **Note on naming:** T1046 was previously called **"Network Service Scanning"** and has since been **renamed to "Network Service Discovery"** in the ATT&CK framework. The ID (**T1046**) and the underlying behaviour are the same — only the name changed to better reflect that attackers may discover services through more than just scanning (e.g., querying APIs, reading config files, using discovery tools).
- **Common tools and methods used by attackers:**
  - `nmap`, `masscan`, `zmap` — port scanners
  - `netdiscover`, `arp-scan` — host discovery
  - `enum4linux`, `smbclient`, `rpcclient` — SMB/RPC enumeration
  - `snmpwalk`, `ldapsearch` — service enumeration
  - `curl`, `Invoke-WebRequest`, PowerShell discovery cmdlets
- **What the defender sees:** A burst of connection attempts to many ports from one source, often with SYN-only traffic, and often followed by a short delay and then targeted exploitation.

---

## 2. Why T1046 matters to a SOC analyst

- It's usually the **first visible phase** of an attack — good detection here can stop the attacker before exploitation.
- It's **high-volume and noisy**, so it's easy to detect if rules exist — but also easy to drown in if not tuned.
- It **overlaps with legitimate admin/scanning activity** — vulnerability scanners, asset management tools, and IT health checks look identical to attacker scans. So context matters.
- It **feeds threat hunting** — you can pivot from a T1046 alert to see what the source host did before and after the scan.
- Detecting T1046 early gives the SOC **time to block the source** before it becomes a breach.

---

## 3. How Snort Rules Map to MITRE Techniques

- Snort itself doesn't "know" about MITRE — mapping happens **by tagging rules** with the ATT&CK technique they detect.
- There are **three ways** to map Snort rules to ATT&CK:
  - **Metadata tag in the rule** — add a `metadata:` option containing the technique ID.
  - **Rule comment / naming convention** — include the technique in `msg` or in a comment above the rule.
  - **External mapping file** — a CSV/YAML that maps Snort SIDs to ATT&CK techniques (common in mature SOCs and used by tools like **SOC Prime**, **Atomic Red Team**, and **Sigma** conversions).
- **Example metadata tag inside a Snort rule:**
  ```snort
  metadata:mitre_technique T1046, mitre_tactic discovery;
\ ```

# Why this matters:
- The SOC can report detection coverage per ATT&CK technique.
- Analysts can search by technique when responding to an incident ("do we have detections for T1046?").
- It helps with purple teaming — you check which techniques are covered and which aren't.
- It aids threat-informed defense — prioritize detection engineering for techniques that threat actors targeting your industry use.

# Mapping table (example techniques → Snort detection ideas): 

ATT&CK ID	| Technique |	What a Snort rule would detect
T1046 | 	Network Service Discovery	 | Port scan — many SYNs to different ports from one source
T1595 |	Active Scanning |	Web vulnerability scanning — many 404s / odd paths from one IP
T1110 |	Brute Force |	Repeated failed SSH or RDP logins
T1071	 | Application Layer Protocol (C2)	 | Beaconing to known-bad domains over HTTP/S
T1041 | 	Exfiltration Over C2 Channel	 | Large outbound uploads to a suspicious host
T1059	 | Command and Scripting Interpreter | 	PowerShell download cradle in HTTP traffic

4. Example — Snort Rule Detecting T1046 (Network Service Discovery)
   Behaviour to detect: A host sending SYN packets to many different destination ports within a short window — classic port scan.
   Approach: Snort has built-in sfportscan preprocessor, but you can also write an explicit rule. In practice, most SOCs enable the portscan preprocessor for broad detection, then use rules for specific scan patterns.

   Example 1 — Snort rule matching a TCP SYN scan pattern (explicit)
    ```snort 
     alert tcp $EXTERNAL_NET any -> $HOME_NET any ( \
    msg:"MITRE T1046 - Network Service Discovery - TCP SYN scan"; \
    flags:S; \
    threshold:type both, track by_src, count 20, seconds 10; \
    classtype:attempted-recon; \
    metadata:mitre_technique T1046, mitre_tactic discovery; \
    sid:1000101; rev:1; \
     )
    ```

   What it does:
     Alerts when a single source IP sends 20 or more SYN packets in 10 seconds to internal hosts. That's a classic port scan.
   Metadata tag:
       metadata:mitre_technique T1046, mitre_tactic discovery; explicitly maps this rule to the technique and tactic

  Example 2 — Snort sfportscan preprocessor (broad) 
   
     preprocessor sfportscan: proto { all } \
    scan_type { portscan, ipscan } \
    sense_level { medium } \
    logfile { portscan.log }
    
What it does: 
   Detects several types of port scans (TCP, UDP, IP, ICMP) using Snort's built-in heuristics. This is the standard way to detect T1046 in Snort and produces alerts that can be tagged with the technique in the SIEM.
  
Example 3 — nmap SYN scan from external source    
  ```   
alert tcp $EXTERNAL_NET any -> $HOME_NET any ( \
    msg:"MITRE T1046 - nmap-style SYN scan to multiple internal ports"; \
    flags:S,12; \
    window:1024; \
    threshold:type both, track by_src, count 15, seconds 5; \
    classtype:attempted-recon; \
    metadata:mitre_technique T1046, mitre_tactic discovery; \
    sid:1000102; rev:1; \
)
```

What it does:
  Matches SYN packets with a small default window (classic nmap fingerprint), and triggers when one source sends 15+ in 5 seconds.

# Key Takeaways

T1046 = Network Service Discovery — attackers mapping live hosts and open services. Previously called "Network Service Scanning", now renamed to reflect all forms of service discovery.

It's a Discovery-phase technique (TA0007) — often the first visible stage of an attack.

Snort rules can be mapped to MITRE by adding a metadata: option with the technique ID, or via an external SID→technique mapping.

Mapping rules to ATT&CK helps the SOC measure detection coverage, prioritize detection engineering, and support purple team exercises.

For T1046, use Snort's sfportscan preprocessor plus targeted rules with threshold on SYN flags — and tag them with mitre_technique T1046.
 
