# Hands-On Task — Set Up Snort and Write Starter Rules

## Task Info
- **Task:** Install Snort, configure HOME_NET, write custom rules, and test alerts.
- **Environment:** Kali Linux VM (Attacker/Target: Localhost)
- **Date Completed:** 2026-11-06

## Steps Performed

### 1. Installation and Verification
- Updated the package repository and installed Snort:
  ```bash
  sudo apt update
  sudo apt install snort -y
  ```

* Verified the network interface name using ip a (Result: eth0).

### 2. Configuration (HOME_NET)
The default Snort configuration file was set up during installation.

HOME_NET was configured to represent the local subnet, and EXTERNAL_NET was set to !$HOME_NET.


 ### 3. Writing Local Rules
Created and edited the local rules file: /etc/snort/rules/local.rules.

Added the following 3 starter rules:
```
 alert icmp any any -> $HOME_NET any (msg:"ICMP Ping Detected"; sid:1000001; rev:1;)
 alert tcp any any -> $HOME_NET 22 (msg:"SSH Connection Attempt"; sid:1000002; rev:1;)
 alert tcp any any -> $HOME_NET 80 (msg:"HTTP Traffic"; sid:1000003; rev:1;)
 ```

### 4. Configuration Test
Attempted to validate the configuration using:   
      ``
      sudo snort -T -i eth0 -c /etc/snort/rules/local.rules
      ```
### 5. Running Snort and Generating Alerts
Attempted to start Snort in console mode to view live alerts:
    ```
    sudo snort -A console -q -i eth0 -c /etc/snort/rules/local.rules
    ```
Generated test traffic using localhost: 
  ```
    ping -c 4 127.0.0.1
    curl http://127.0.0.1
  ```
Screenshot 
  
  ### Key Takeaways / Reflections
Rule Syntax: I learned the basic structure of a Snort rule: [Action] [Protocol] [Source] [Port] -> [Destination] [Port] (Options). The sid must be unique for local rules (1000001, 1000002, etc.).

Environment Issues: During testing in the Kali Linux VM, Snort encountered a syntax error (unexpected near 'icmp') due to invisible character encoding issues in the terminal/nano editor.

Troubleshooting: I learned that configuration files must be strictly formatted with no trailing spaces or hidden characters. In a real production environment, configuration management tools (like Ansible) are used to deploy these rules consistently.