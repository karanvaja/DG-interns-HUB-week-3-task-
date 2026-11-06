# Hands-On Task — Explore Linux Auditing with auditd

## Task Info
- **Task:** Install, configure, and investigate Linux commands using auditd.
- **Environment:** Kali Linux VM
- **Date Completed:** 2026-11-06

## Steps Performed

### 1. Installation and Setup
- Verified that the `auditd` service is running:
  ```bash
  sudo systemctl start auditd
  ```

2. Creating an Audit Rule
* By default, Kali Linux does not log every command to save disk space. I manually created a rule to track all command executions (execve system calls):

```
sudo auditctl -a always,exit -F arch=b64 -S execve -k my_commands
```

(The -k my_commands is a custom key/tag that helps us easily search for these specific events later).

3. Executing an Audited Command
* Ran a test command to generate an audit trail:

``` 
ls -la
 ```
4. Searching the Audit Log
Used ausearch to find the events associated with my custom key:
```
sudo ausearch -k my_commands
```

Screenshot 
`![Auditd Ausearch Output](image/Auditd_Task.png)`


Audit Log Fields Documentation
When analyzing the output of ausearch, several key fields are available. Here is what they mean and why they are useful to a SOC Analyst:

Field	| Example |	What it means	 |Why it's useful to a SOC Analyst

type |	type=EXECVE | 	The type of audit record. EXECVE means a command was executed. SYSCALL shows the system call used. CWD shows the directory. |	Helps analysts filter logs quickly (e.g., looking only for process execution vs file access).
msg	 |audit(1793941283.284:130)	 | The audit message ID and timestamp.	 | Provides the exact time of execution, which is critical for building a timeline of an attack.
exe | 	exe="/usr/bin/cut"	 | The full path of the executable that was run. |	Shows what program was run. If an attacker runs a script from /tmp/, it will be visible here.
a0, a1, a2	 | a0="cut", a1="-d"	 | The arguments passed to the command. | 	Extremely important. It shows exactly what the user was trying to do (e.g., adding a user, downloading a file).
uid / gid | 	uid=1000 gid=1000 | 	The User ID and Group ID that executed the command. 0 means root. |	Helps identify who ran the command. If a normal user (uid=1000) is running administrative commands, it's a red flag.
pid / ppid |	pid=6798 ppid=6800 |	Process ID and Parent Process ID. |	Allows analysts to build a process tree. For example, if bash spawned nc (netcat), the ppid helps trace it back.
cwd	| cwd="/home/karan"	| Current Working Directory.	 | Shows the directory from where the command was executed, giving context to the attacker's location on the filesystem.
key	|key="my_commands"	|The custom tag associated with the audit rule.|	If a SOC engineer set up a specific rule, the key helps group those specific security events.

* Personal Notes / Reflections
I learned that ausearch is much easier to read than manually digging through /var/log/audit/audit.log.

The EXECVE message type is extremely powerful for detecting malicious command lines. In a real SOC, this is mapped to SIEMs like Wazuh to detect brute-force scripts or reverse shells.

I noticed the SYSCALL record right before EXECVE. The SYSCALL record shows the low-level system call (like execve), and the EXECVE record shows the actual human-readable arguments.