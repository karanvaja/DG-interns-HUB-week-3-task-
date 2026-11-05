# Hands-On Task — Installing Sysmon with SwiftOnSecurity Config

## Task Info
- **Task:** Install and configure Sysmon on a Windows Host.
- **Environment:** Local Windows 11 Machine (Host OS)
- **Date Completed:** 2026-11-05

## Objective
- To install Microsoft Sysmon for deep endpoint telemetry.
- To apply a widely-used community configuration (SwiftOnSecurity) to filter noise and capture critical events (Process Creation, Network Connections, DNS Queries, etc.).

## Steps Performed
1. **Preparation:** Created a directory `C:\Sysmon` to store the executable and configuration file.
2. **Download:** 
   - Downloaded the Sysmon ZIP from the official Microsoft Sysinternals page and extracted its contents into `C:\Sysmon`.
   - Downloaded the `sysmonconfig-export.xml` (SwiftOnSecurity config) from GitHub and saved it in the same folder.
3. **Installation:** 
   - Opened Command Prompt as Administrator.
   - Navigated to the directory: `cd C:\Sysmon`
   - Ran the installation command: 
    
     ```
     cmd
     sysmon.exe -accepteula -i sysmonconfig-export.xml

     The command executed successfully, validating the configuration and installing the service. 
    
    ```


4. Verification:

Opened Windows Event Viewer (eventvwr.msc).

Navigated to Applications and Services Logs -> Microsoft -> Windows -> Sysmon -> Operational.

Confirmed that events are actively being logged. Noticed Event ID 1 (Process Create), Event ID 3 (Network Connection), and Event ID 22 (DNS Query).

`![Sysmon Operational Log](./image/Sysmon_Installation_Screenshot.png)`


# Key Takeaways / Reflections

Telemetry is crucial: Standard Windows logs are not enough for a SOC. Sysmon provides the detailed context (full command lines, parent processes, hashes) needed to detect modern attacks like fileless malware and LOLBins.

Configuration matters: Without a config like SwiftOnSecurity, Sysmon would generate a massive amount of noise. The XML config file tells Sysmon exactly what to include and exclude, making the logs manageable for SIEM ingestion.

Log Location: All Sysmon logs are stored in the Event Viewer under the Operational channel. In a real SOC, these logs are forwarded to a SIEM (like Splunk or Wazuh) using an agent like Splunk Universal Forwarder or Wazuh Agent. 