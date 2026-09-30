
# 🔍 Lab 1: Real-time Process Analysis and Socket Auditing on Linux

## 🎯 Objective
The objective of this practical lab is to audit active network connections on a Linux system to identify anomalies and simulate a forensic analysis of processes (*Threat Hunting*). The technical procedure for tracking a suspicious Process ID (PID), locating its physical path on the disk and validating its structural integrity to rule out persistence vectors or malware camouflage is documented.

## 🧰 Tools and Commands Used
* **`ss -abnop`**: For advanced analysis and auditing of sockets (network, interfaces and associated processes).
* **`ls -l /proc/[PID]/exe`**: To locate the actual physical path of the running binary by inspecting the kernel’s symbolic link.
  * **`file`**: To analyse file headers and verify the internal structural signature of the executable.
* **OSINT / CTI sources**: Cyber Threat Intelligence methodologies for infrastructure enrichment and reputation analysis.

----


## 🚀 Enforcement and Investigation Process
### 1. Audit of Network Connections and Active Sockets
A process maintaining active network connections on the system was analysed. Using advanced socket analysis tools, the process ID (PID) was traced until the physical path of the executable binary file on the disk was located and its internal structure validated using THESE COMMANDS:

```shell
ss -abnop
sudo ls -l /proc/1509/exe
sudo file /usr/lib/mate-panel/wnck-applet
```

![[Pasted image 20260812164415.png]]

When evaluating the output, the analysis reveals two types of behaviour that are critical to the audit:
* **Local Unix sockets (`u_str`, `u_dgr`):** Legitimate system processes operating internally are identified, notably `wnck-applet` under **PID 1509**, alongside essential services such as `systemd` (PID 1132) and `dbus-daemon` (PID 1387).
  
  * **Network Connections and Interfaces (`udp`, `tcp`):** Active outbound connections to external servers have been detected (IP `9.9.9.9` via ephemeral ports `49605` and `60901`). Furthermore, a local service is observed waiting for traffic in an unusual manner on **TCP port 2222** (`*:2222`), which triggers a security alert as this is an alternative port commonly exploited for *backdoors* or hidden SSH sessions.

### 2. Enrichment with Threat Intelligence (IoC Investigation)
As part of the Threat Intelligence workflow, we extract the Indicators of Compromise (IoCs) detected during the socket audit for further analysis and correlation:

* **Destination IP:** `9.9.9.9`
* **Destination port:** `2222`

**CTI procedure applied:** 

1. **Reputation Check:** In a real-world operational scenario, we cross-check the suspicious IP address and port against Threat Intelligence platforms (such as *VirusTotal*) to verify whether they are linked to Command and Control (C2) servers or botnet nodes, or whether they belong to a legitimate infrastructure (in this particular case, the Quad9 public DNS service).
   
   2. **Threat Contextualisation:** We cross-reference the unusual port `2222` with databases of known vulnerabilities to identify which families of malware or persistence Trojans typically use this port to evade conventional perimeter defences.

### 3. Tracing the Physical Path of the Process (Forensic Logic)
To verify the origin of the process associated with the suspect port, we investigate its virtual directory in the kernel’s file system:

```bash
sudo ls -l /proc/1509/exe
```
 **💡 Learning Note:** It is common for junior analysts to be confused when they see the word `exe` in the path `/proc/[PID]/exe`, mistakenly associating it with Windows executables. In the Linux architecture, `exe` is simply the name of the generic symbolic link used by the kernel to point to the actual native binary on the disk.

### 4. Structural Validation and Binary Integrity

```bash
sudo file /usr/lib/mate-panel/wnck-applet
```

**Evaluation Criteria for the `file` command in Incident Response:**
* **Legitimate Native Binary:** If the command returns an **ELF** format (`ELF 64-bit LSB shared object` or `executable`), we confirm that it is a native binary legitimately compiled for Linux.
* **Hidden Scripts:** If the output shows `Python script` or `ASCII text executable` in a critical path where the system expects a binary, it is classified as a spoofing anomaly.
  * **Malware / Trojan Pattern:** A definitive indicator of compromise (IoC) occurs if the `file` command confirms the structure of an ELF executable binary, but its physical path is located in temporary or volatile directories such as `/tmp/...` or `/dev/shm/`. This behaviour reveals a classic pattern of malware attempting to evade traditional persistence mechanisms.

----

## 🛡️ Proper Defence and Mitigation (SOC Playbook)
If any suspicious open ports are detected (such as the `2222` analysed), the incident response workflow dictates:
1. Immediately run the `file` command on the PID link to examine the header.
2. Verify that the executable resides exclusively in secure, read-only system directories (such as `/usr/lib/` or `/usr/sbin/`).
3. If the binary is running from temporary directories, the machine is isolated from the network, a memory dump of the process is taken for analysis in a sandbox, and the compromised process is immediately terminated (*kill*) using its PID.
