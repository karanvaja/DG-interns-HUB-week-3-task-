# Q6 — Signature-Based vs Anomaly-Based Detection

## 1. What is Signature-Based Detection?

- **Signature-based detection** works by matching observed activity against a **known pattern (signature)** of malicious behaviour.
- A signature can be a **file hash**, a **byte string** in a packet payload, a **specific command line**, a **registry key**, or a **YARA rule**.
- If the observed traffic/process/file matches a known signature, an alert is raised. If the attacker changes anything (e.g., recompiles malware), the signature **no longer matches**.
- It's the **oldest and most common** detection method — antivirus, Snort rules, and many SIEM correlation rules are signature-based.
- **Strength:** very **low false positives** and **fast** to run.
- **Weakness:** **cannot detect unknown or modified threats** (zero-days, polymorphic malware, custom tooling).

---

## 2. What is Anomaly-Based Detection?

- **Anomaly-based detection** establishes a **baseline of "normal" behaviour** (for a user, host, network segment, or application) and then alerts on **deviations** from that baseline.
- Instead of asking *"does this match a known bad pattern?"* it asks *"does this look different from what we usually see?"*
- Baselines can be built using:
  - **Statistical models** (mean, standard deviation).
  - **Machine learning** (clustering, autoencoders, isolation forests).
  - **Rule-based thresholds** (e.g., "user logs in from more than 3 countries in a day").
- **Strength:** can catch **unknown and novel attacks** (zero-days, insider threats, custom malware).
- **Weakness:** **high false positive rate** (because "different" isn't always "malicious") and **hard to tune** — requires good baseline data.

---

## 3. Signature-Based vs Anomaly-Based — Comparison Table

| Aspect | Signature-Based Detection | Anomaly-Based Detection |
|--------|----------------------------|--------------------------|
| **How it detects** | Matches known bad patterns | Detects deviations from a learned baseline |
| **Known threats** | Excellent | Good (if the baseline was healthy) |
| **Unknown / zero-day threats** | Poor — misses them | Strong — can catch novel behaviour |
| **False positives** | Low (false positives usually mean a poorly written rule) | High — baseline drift and legitimate change trigger alerts |
| **False negatives** | High for novel attacks | Lower, but depends on baseline quality |
| **Setup effort** | Low — deploy rules | High — needs data collection, training, tuning |
| **Maintenance** | Update signatures regularly | Retrain/refresh baselines periodically |
| **Explainability** | Easy — analyst can read the rule | Harder — "the model said so" |
| **Performance** | Fast | Slower (needs ML models and pipelines) |
| **Attackers bypass by** | Slight changes to malware or behaviour | "Living off the land" (using only normal tools) |
| **Typical tools** | Antivirus, Snort/Suricata rules, YARA, Sigma rules | UEBA, ML-based EDR, DNS anomaly detection, network flow analytics |
| **Best for** | Known threats, compliance, quick wins | Insider threats, novel malware, lateral movement, data exfiltration |

---

## 4. SOC Examples of Each

- **Signature-based examples in a SOC:**
  - **Snort rule** matching `UNION SELECT` in HTTP traffic → SQL injection alert.
  - **YARA rule** matching the hash of a known ransomware sample.
  - **Sigma/SIEM rule** alerting on `mimikatz.exe` being executed (by name or hash).
  - **Antivirus** blocking a known phishing attachment hash.
  - **IOC match** — alerting when an endpoint contacts a known C2 IP from a threat-intel feed.
  - **Windows Event ID 4688** rule: "powershell.exe -EncodedCommand" → alert on encoded PowerShell.

- **Anomaly-based examples in a SOC:**
  - **UEBA alert:** a finance user who normally logs in from Mumbai suddenly logs in from Russia at 3 AM.
  - **Network flow anomaly:** a workstation that usually sends 10 MB/day suddenly uploads 5 GB overnight → possible data exfiltration.
  - **DNS anomaly:** a host making hundreds of DNS requests to random subdomains → possible DGA / DNS tunneling.
  - **Process behaviour anomaly:** `notepad.exe` (a GUI app) suddenly spawning `cmd.exe` with network connections → possible malware.
  - **Login anomaly:** a service account that has never logged in interactively suddenly logs into 15 servers in 5 minutes → lateral movement.
  - **EDR behavioural alert:** `winword.exe` → `powershell.exe` → `curl.exe` chain → possible phishing payload.

- **In practice, SOCs combine both:**
  - **Signature-based rules** catch the well-known stuff fast and cleanly.
  - **Anomaly-based detection** catches the "unknown unknowns" and behaviour that no rule has ever described.
  - The **best SOCs use both together** — signatures for precision, anomalies for coverage — and tune anomaly models with feedback from the analyst team.

---

## Key Takeaways

- **Signature = match known bad. Anomaly = detect "different from normal".**
  
- Signatures are **fast, precise, low false positives**, but **blind to novel threats**.
  
- Anomalies catch **unknown threats**, but bring **noise, tuning pain, and explainability challenges**.
  
- Good SOCs **layer both** — and use **endpoint, network, identity and cloud telemetry** to feed both detection styles.
  
- When anomaly detection fires, an analyst's job is to **ask: is this new normal, or actual malicious behaviour?** — that's the tuning feedback loop.
