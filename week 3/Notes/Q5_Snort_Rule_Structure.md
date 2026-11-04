# Q5 — Structure of a Snort Rule: Header, Options, and Examples

## 1. What is a Snort Rule?

- A **Snort rule** is a single line of text that tells Snort **what traffic to look for and what to do when it finds it**.
- Every rule has **two main parts**:
  - **Rule Header** — defines the *action*, *protocol*, *source/destination IP and port*, and *direction*.
  - **Rule Options** — defines *what specifically to match* (payload, flags, flow, metadata, etc.), enclosed in parentheses.
- Basic format:       <action> <protocol> <src_ip> <src_port> -> <dst_ip> <dst_port> ( <options> )

- - Rules live in files like `/etc/snort/rules/local.rules` and are loaded by Snort at startup.
- A SOC analyst writes **custom local rules** for environment-specific threats and also relies on **community/official rulesets** (Emerging Threats, Snort VRT).

---

## 2. Breakdown of the Rule Header

- The **header** answers: *What action? What protocol? From where? To where? In which direction?*
- Format:     action protocol src_ip src_port direction dst_ip dst_port

- - **Parts of the header:**

| Part | Meaning | Examples |
|------|---------|----------|
| **Action** | What Snort does on match | `alert`, `log`, `pass`, `drop`, `reject` |
| **Protocol** | Which protocol to inspect | `tcp`, `udp`, `icmp`, `ip` |
| **Source IP** | Where the traffic comes from | `any`, `10.0.0.0/8`, `$HOME_NET`, `!192.168.1.5` |
| **Source Port** | Source port to match | `any`, `1024:`, `:1024`, `443` |
| **Direction** | Traffic direction | `->` (one-way), `<>` (both ways) |
| **Destination IP** | Where the traffic goes | `any`, `$EXTERNAL_NET`, `192.168.1.0/24` |
| **Destination Port** | Destination port to match | `any`, `80`, `22`, `!443` |

- **Variables** like `$HOME_NET` and `$EXTERNAL_NET` are defined in `snort.conf` — so the same rule works across different environments.
- **Example header only:**
        alert tcp   EXTERNALnETany- > HOME_NET 22
  *"Alert on any TCP traffic coming from outside to internal SSH port 22."*

---

## 3. Breakdown of the Rule Options

- **Options** are inside parentheses `( ... )` and are separated by **semicolons `;`**.
- They tell Snort **exactly what to match** and **how to describe the alert**.
- **Types of options:**
- **General options** — metadata for the alert (msg, sid, rev, classtype, reference, priority).
- **Payload options** — match content inside the packet (content, nocase, depth, offset, pcre).
- **Non-payload options** — match IP/TCP/UDP headers, flags, flow, etc. (flags, flow, ttl, id, ip_proto).
- **Post-detection options** — decide what to do after a match (threshold, tag, replace).

- **Common rule options table:**

| Option | Type | What it does | Example |
|--------|------|--------------|---------|
| **msg** | General | Alert message shown to the analyst | `msg:"Possible SSH brute force";` |
| **sid** | General | Unique rule ID (Snort ID) | `sid:1000001;` |
| **rev** | General | Revision number (increment on edits) | `rev:1;` |
| **flags** | Non-payload | Match TCP flags (S, A, F, R, P, U) | `flags:S;` (SYN packet) |
| **flow** | Non-payload | Match flow state and direction | `flow:to_server,established;` |
| **threshold** | Post-detection | Rate-limit alerts (e.g., 5 per 60s) | `threshold:type both, track by_src, count 5, seconds 60;` |
| **classtype** | General | Category of the rule (classification) | `classtype:attempted-recon;` |
| **content** | Payload | Match a byte string in the payload | `content:"GET /admin";` |
| **nocase** | Payload modifier | Case-insensitive content match | `content:"USER"; nocase;` |
| **pcre** | Payload | Match a regular expression | `pcre:"/User-Agent\:[^\n]*sqlmap/i";` |
| **reference** | General | Link to a CVE, URL, or external ID | `reference:cve,2021-44228;` |
| **priority** | General | Numeric priority for the alert | `priority:1;` |
| **depth / offset** | Payload modifier | Where in the packet to start/stop matching | `content:"POST"; depth:4;` |

---

## 4. Full Example Rule + Walkthrough

**Example: SSH brute-force attempt detection**
```snort
     alert tcp $EXTERNAL_NET any -> $HOME_NET 22 ( \
         msg:"Possible SSH brute force from external host"; \
         flow:to_server,established; \
         flags:S; \
         threshold:type both, track by_src, count 5, seconds 60; \
         classtype:attempted-recon; \
         sid:1000001; rev:1; \
   ```

# Line-by-line walkthrough:

 | Part	| Explanation
alert tcp	| Action = alert, protocol = TCP
$EXTERNAL_NET any -> |	Source = any port from external network
$HOME_NET 22	 |Destination = internal SSH (port 22)
msg:"..." |	Human-readable alert name
flow:to_server,established	 | Match traffic flowing from client to server, established state
flags:S	 | Match packets with the SYN flag set
threshold:type both, track by_src, count 5, seconds 60	 | Only alert if ≥ 5 SYNs from the same source in 60 seconds
classtype:attempted-recon |	Categorize as reconnaissance
sid:1000001; rev:1;	 | Unique ID and revision

# Example 2: Detect SQL injection attempt in HTTP
 ```  
   alert tcp $EXTERNAL_NET any -> $HOME_NET 80 ( \
    msg:"SQL injection attempt - UNION SELECT"; \
    flow:to_server,established; \
    content:"UNION"; nocase; \
    content:"SELECT"; nocase; distance:0; \
    classtype:web-application-attack; \
    sid:1000002; rev:1; \
)
```

# Example 3: Detect PowerShell download cradle (LOLBin behaviour over HTTP)
 ``` 
   alert tcp $HOME_NET any -> $EXTERNAL_NET any ( \
    msg:"PowerShell DownloadString C2 pattern"; \
    flow:to_client,established; \
    content:"IEX"; nocase; \
    content:"DownloadString"; nocase; \
    classtype:trojan-activity; \
    sid:1000003; rev:1; \
)
```
Key Takeaways
A Snort rule = Header (action/proto/src/dst/direction) + Options (what to match + metadata).
The header decides what happens on match; the options decide what actually triggers the match.
sid must be unique — local rules typically use 1000000+ range.
Use threshold to suppress noisy alerts; use flow and flags to narrow matching.
Always increment rev when you edit a rule, so you can track changes.
Well-written rules are specific, low-false-positive, and mapped to a real threat (often tied to a MITRE technique).


  
