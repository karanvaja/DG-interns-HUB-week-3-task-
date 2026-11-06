# Hands-On Task — Advanced Snort Rule (Nmap SYN Scan & MITRE T1046)

## Task Info
- **Task:** Write a Snort rule to detect an Nmap SYN scan and map it to MITRE ATT&CK T1046 (Network Service Discovery).
- **Environment:** Kali Linux VM (Localhost testing)
- **Date Completed:** 2026-11-06

## Objective
- To write a threshold-based Snort rule to detect port scanning.
- To map the detection to a known MITRE ATT&CK technique (T1046).
- To test the rule and observe alert generation.

## Steps Performed

### 1. Adding the Advanced Rule
- Edited `/etc/snort/rules/local.rules` and added the following rule along with the previous 3 starter rules:
  ```snort
  alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"MITRE T1046 - Nmap SYN Port Scan Detected"; flags:S; threshold:type threshold, track by_src, count 20, seconds 1; classtype:attempted-recon; metadata:mitre_technique T1046; reference:url,attack.mitre.org/techniques/T1046/; sid:1000004; rev:1;)
```

```



### 2. Rule Breakdown (Understanding the Options)
flags:S: Matches TCP packets with only the SYN flag set. This is the signature of an Nmap -sS (half-open) scan.

threshold:type threshold, track by_src, count 20, seconds 1: Prevents alert fatigue. It only triggers an alert if 20 SYN packets are sent from the same source IP within 1 second.

classtype:attempted-recon: Categorizes the alert as reconnaissance (information gathering).

metadata:mitre_technique T1046: Explicitly tags the alert with the MITRE technique ID for SIEM correlation.

reference:url: Provides a link to the official MITRE ATT&CK page for T1046.


### 3. Starting Snort
Attempted to start Snort in console mode:

``` sudo snort -A console -q -i eth0 -c /etc/snort/rules/local.rules
```
(Note: Encountered an environment-specific invisible character error in Kali Linux. The troubleshooting steps are documented in the Task 4 writeup).

### 4. Executing the Nmap Scan (Traffic Generation)
In a new terminal, ran an Nmap SYN scan against localhost to generate the required traffic:

```
 sudo nmap -sS 127.0.0.1 
 ```

* The scan generated multiple SYN packets rapidly, which is the exact behavior the rule was designed to catch. 



### Key Takeaways / Reflections
Thresholding is essential: Without the threshold option, a single scan would generate thousands of alerts, overwhelming the SOC. Setting a limit (e.g., 20 per second) helps focus on the actual scan behavior rather than individual packets.

MITRE Mapping: Adding metadata to rules helps analysts understand why the alert matters. T1046 is usually the very first step an attacker takes (Discovery phase), so catching it early is vital.

Localhost Testing: Using 127.0.0.1 for the Nmap scan was a smart way to test the rule without needing a second VM or risking scanning external networks.