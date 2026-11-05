# TryHackMe — Snort

## Lab Info
- **Platform:** TryHackMe
- **Room Name:** Snort
- **Difficulty:** Easy / Medium
- **Time to Complete:** ~120 minutes
- **Date Completed:** 2026-11-05

## Objective
- Understand Intrusion Detection Systems (IDS) and Intrusion Prevention Systems (IPS).
- Learn the three main modes of Snort: Sniffer, Packet Logger, and NIDS/NIPS.
- Understand the structure of Snort rules (Header and Options).
- Learn how to read, analyze, and filter PCAP files using Snort.

---

## Task Writeups & Answers

### Task 1: Introduction
- **Overview:** Introduction to Snort, its history (created by Martin Roesch, maintained by Cisco Talos), and its capabilities as a rule-based NIDS/NIPS.
- **Answer:** No answer needed.

### Task 2: Interactive Material and VM
- **Overview:** Setting up the virtual environment. The lab uses a traffic generator script to simulate different types of traffic (ICMP, HTTP, etc.) so Snort has data to process.
- **Answer:** `Too Easy!` (Output of the `./easy.sh` command).

### Task 3: Introduction to IDS/IPS
- **Overview:** Differentiating between IDS (passive, detects and alerts) and IPS (active, detects and blocks). Also covered detection techniques: Signature-based, Behaviour-based, and Policy-based. 
- **Questions & Answers:**
  - **Which IDS or IPS type can help you stop the threats on a local machine?**
    - **Answer:** `HIPS`
  - **Which IDS or IPS type can help you detect threats on a local network?**
    - **Answer:** `NIDS`
  - **Which IDS or IPS type can help you detect the threats on a local machine?**
    - **Answer:** `HIDS`
  - **Which IDS or IPS type can help you stop the threats on a local network?**
    - **Answer:** `NIPS`
  - **Which described solution works by detecting anomalies in the network?**
    - **Answer:** `NBA`
  - **According to the official description of the snort, what kind of NIPS is it?**
    - **Answer:** `full-blown`
  - **NBA training period is also known as ...**
    - **Answer:** `baselining`

### Task 4: First Interaction with Snort
- **Overview:** Checked the Snort version (`-V`) and tested the configuration file using the `-T` parameter.
- **Questions & Answers:**
  - **Run the Snort instance and check the build number.**
    - **Answer:** `149`
  - **Test the current instance with "/etc/snort/snort.conf" file and check how many rules are loaded with the current build.**
    - **Answer:** `4151`
  - **Test the current instance with "/etc/snort/snortv2.conf" file and check how many rules are loaded with the current build.**
    - **Answer:** `1`

### Task 5: Operation Mode 1: Sniffer Mode
- **Overview:** Used Snort as a packet sniffer (like tcpdump) using parameters like `-v` (verbose), `-d` (payload), `-e` (link-layer headers), and `-X` (full packet in HEX).
- **Answer:** `No answer needed` (Practical task).

### Task 6: Operation Mode 2: Packet Logger Mode
- **Overview:** Logged packets to disk using the `-l` parameter. Learned about output directories, file ownership (needs `sudo`), and the difference between binary tcpdump format and ASCII format (`-K ASCII`). Used `-r` to read the logs back.
- **Questions & Answers:**
  - **Investigate the traffic with the default configuration file with ASCII mode. What is the source port used to connect port 53?**
    - **Answer:** `3009`
  - **Read the snort.log file with Snort; what is the IP ID of the 10th packet?**
    - **Answer:** `49313`
  - **Read the "snort.log.1640048004" file with Snort; what is the referer of the 4th packet?**
    - **Answer:** `http://www.ethereal.com/development.html`
  - **Read the "snort.log.1640048004" file with Snort; what is the Ack number of the 8th packet?**
    - **Answer:** `0x38AFFFF3`
  - **Read the "snort.log.1640048004" file with Snort; what is the number of the "TCP port 80" packets?**
    - **Answer:** `41`

### Task 7: Operation Mode 3: IDS/IPS
- **Overview:** Ran Snort in IDS/IPS mode using configuration files (`-c`). Explored alert modes (`-A`): `console`, `cmg`, `fast`, `full`, and `none`. Also learned about background mode (`-D`).
- **Questions & Answers:**
  - **Investigate the traffic with the default configuration file. What is the number of the detected HTTP GET methods?**
    - **Answer:** `2`
  - **You can practice the rest of the parameters by using the traffic-generator script.**
    - **Answer:** `No answer needed`

### Task 8: Operation Mode 4: PCAP Investigation
- **Overview:** Used Snort to analyze PCAP files using `-r` (single PCAP) and `--pcap-list` (multiple PCAPs). Used `--pcap-show` to identify which PCAP generated which alert.
- **Questions & Answers:**
  - **Investigate the mx-1.pcap file with the default configuration file. What is the number of the generated alerts?**
    - **Answer:** `170`
  - **Keep reading the output. How many TCP Segments are Queued?**
    - **Answer:** `18`
  - **Keep reading the output. How many "HTTP response headers" were extracted?**
    - **Answer:** `3`
  - **Investigate the mx-1.pcap file with the second configuration file. What is the number of the generated alerts?**
    - **Answer:** `68`
  - **Investigate the mx-2.pcap file with the default configuration file. What is the number of the generated alerts?**
    - **Answer:** `340`
  - **Keep reading the output. What is the number of the detected TCP packets?**
    - **Answer:** `82`
  - **Investigate the mx-2.pcap and mx-3.pcap files with the default configuration file. What is the number of the generated alerts?**
    - **Answer:** `1020`

### Task 9: Snort Rule Structure
- **Overview:** Broke down Snort rules into two parts: **Rule Header** (Action, Protocol, Source IP/Port, Direction, Destination IP/Port) and **Rule Options** (msg, sid, rev, content, flags, etc.). Created local rules to filter specific traffic patterns.
- **Questions & Answers:**
  - **Use "task9.pcap". Write a rule to filter IP ID "35369" and run it against the given pcap file. What is the request name of the detected packet?**
    - **Answer:** `TIMESTAMP REQUEST`
  - **Clear the previous alert file and comment out the old rules. Create a rule to filter packets with Syn flag and run it against the given pcap file. What is the number of detected packets?**
    - **Answer:** `1`
  - **Clear the previous alert file and comment out the old rules. Write a rule to filter packets with Push-Ack flags and run it against the given pcap file. What is the number of detected packets?**
    - **Answer:** `216`
  - **Clear the previous alert file and comment out the old rules. Create a rule to filter UDP packets with the same source and destination IP and run it against the given pcap file. What is the number of packets that show the same source and destination address?**
    - **Answer:** `7`
  - **Case Example - An analyst modified an existing rule successfully. Which rule option must the analyst change after the implementation?**
    - **Answer:** `rev`

### Task 10: Snort2 Operation Logic: Points to Remember
- **Overview:** Summarized the main components of Snort (Packet Decoder, Pre-processors, Detection Engine, Logging, Outputs). Explained the different rule sets (Community, Registered, Subscriber) and the main `snort.conf` configuration sections.
- **Answer:** `No answer needed`

### Task 11: Conclusion
- **Overview:** Wrapped up the room. Provided a Snort cheat sheet and next steps (Snort Challenges).
- **Answer:** `No answer needed`

---

## Key Takeaways
- **IDS vs IPS:** IDS is passive (just alerts), IPS is active (drops packets).
- **Snort Modes:** Sniffer (`-v`), Packet Logger (`-l`), and NIDS/NIPS (`-c`).
- **Rule Structure:** `[Action] [Protocol] [Src IP] [Src Port] -> [Dst IP] [Dst Port] (Options)`.
- **SIDs:** Local rules should use `sid: 1000000+`. Always update `rev` when modifying a rule.
- **Read PCAPs:** `snort -r file.pcap` allows you to run IDS rules against recorded traffic, which is highly useful for incident response.

## Commands / Tools Used
```bash
# Version and Config test
snort -V
sudo snort -c /etc/snort/snort.conf -T

# Sniffer modes
sudo snort -v
sudo snort -d
sudo snort -de
sudo snort -X

# Logger modes
sudo snort -dev -l .
sudo snort -dev -K ASCII -l .
sudo snort -r snort.log.1638459842
sudo tcpdump -r snort.log.1638459842 -ntc 10

# IDS mode (alerts)
sudo snort -c /etc/snort/snort.conf -A console
sudo snort -c /etc/snort/snort.conf -A cmg
sudo snort -c /etc/snort/snort.conf -A fast
sudo snort -c /etc/snort/snort.conf -A full
sudo snort -c /etc/snort/snort.conf -A none

# PCAP Investigation
sudo snort -c /etc/snort/snort.conf -r icmp-test.pcap -A console -n 10
sudo snort -c /etc/snort/snort.conf --pcap-list="icmp-test.pcap http2.pcap" -A console
sudo snort -c /etc/snort/snort.conf --pcap-list="icmp-test.pcap http2.pcap" -A console --pcap-show