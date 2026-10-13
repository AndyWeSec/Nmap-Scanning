#!/usr/bin/env python3
"""
Automated GitHub Security Portfolio Repository Compiler
Generates the complete structure, markdown assets, and scripts for the Nmap Scanning Lab.
"""

import os
import stat

def build_repository():
    # 1. Establish directory boundaries for screenshots
    print("[*] Initializing repository asset directories...")
    os.makedirs("screenshots", exist_ok=True)

    # 2. Write the absolute complete README file containing precise figure mappings
    print("[*] Crafting professional portfolio README.md file...")
    readme_content = """# 🛡️ Foundational Network Reconnaissance with Nmap 7.99

A practical study of foundational network reconnaissance techniques using **Nmap 7.99** on a **Kali Linux VM**. This project explores host discovery, port scanning, service versioning, and OS fingerprinting from both offensive and defensive postures.

* **Learning with:** AndrewCyberJones and Empirical Training
* **Lab Engine:** Nmap 7.99 on Kali Linux (hosted on macOS)

---

## 📋 Project Overview
This project covers the four core Nmap commands that form the bedrock of network reconnaissance: finding live hosts on a network, scanning for open ports, identifying the software and versions behind those ports, and guessing the operating system. Each command was executed from a Kali Linux virtual machine, evaluated line-by-line, and mapped out from both sides: what an attacker learns, and how a SOC analyst would spot and use the same activity.

### 🔒 Rules of Engagement
* **Local Scope:** The host discovery scan ran strictly against my own home network (`192.168.1.0/24`).
* **Remote Scope:** The other three scans targeted `scanme.nmap.org` (45.33.32.156), the Nmap project's official test host.
* *Note: The Nmap project explicitly permits scanning this machine. Exploits and DoS attempts are prohibited, and traffic was capped within the suggested limit of roughly a dozen a day.*

---

## 🖥️ Lab Environment

| Component | Detail |
| :--- | :--- |
| **Scanner** | Nmap 7.99 on Kali Linux, running in a virtual machine on my Mac |
| **VM Network Interface** | `eth0` at `192.168.1.126` with netmask `255.255.255.0` (a `/24` network) |
| **Local Target Subnet** | `192.168.1.0/24`, my own home network |
| **Remote Target Host** | `scanme.nmap.org` (45.33.32.156) |

---

## 🛠️ Walkthrough & Execution

### ⚙️ Phase 0: Finding My Network Range
Before scanning my network, I had to find its range. Running `ifconfig` showed `eth0` with `inet 192.168.1.126` and `netmask 255.255.255.0`. A `255.255.255.0` mask is a `/24`, meaning my network range covers 256 addresses (`.0` to `.255`).

```bash
ifconfig eth0
```

---
### 📸 SCREENSHOT POINT 0: LOCAL INTERFACE MAPPING
![Ifconfig Output](screenshots/0_ifconfig_output.png)
*Figure 0: Determining local interface metrics inside the Kali Linux console environment.*
---

### 🔍 Command 1: Host Discovery (Ping Sweep)
```bash
nmap -sn 192.168.1.0/24
```
* **Mechanics:** `nmap` is the scanner, `-sn` means no port scan (checks which hosts are alive), and `192.168.1.0/24` is the range to check. On a local network, Nmap uses ARP requests, which bypass OS firewalls because devices must answer ARP to communicate.
* **Lab Discoveries:** Finished in 2.64 seconds, locating **14 live hosts**. Reading the vendor names provided immediate intelligence:

---
### 📸 SCREENSHOT POINT 1: PING SWEEP DISCOVERIES
![Ping Sweep Output](screenshots/1_ping_sweep.png)
*Figure 1: Executing live network sweep discovery options over the local home environment.*
---

| What Nmap Showed | Likely Meaning / Asset Profile |
| :--- | :--- |
| `192.168.1.1`, vendor Icotera A/S | My home router gateway |
| Several Amazon Technologies devices | Amazon smart speakers or smart home nodes |
| Two Vestel Elektronik devices | Smart TVs |
| Two TP-Link devices (`Tapo_Bulb` / `L510`) | Smart bulbs |
| One Ring device | A doorstep doorbell or security camera |
| Devices with vendor "Unknown" | Mobile devices or laptops utilizing private MAC address randomization |
| Host "Mac" at `0.0002s` latency | My physical host Mac hosting the Kali virtual instance |
| Host "kali" with no MAC shown | The target local VM I was actively scanning from |

* **💥 Attacker View:** The starting blueprint. It uncovers weak smart-home or IoT configurations across an area in seconds.
* **🛡️ Defender View:** Active asset tracking. Defenders catch this pattern when a single device fires sweeps to an entire subnet.

---

### 🚪 Command 2: Basic Port Scan
```bash
nmap scanme.nmap.org
```
* **Mechanics:** Evaluates the **1,000 most common TCP ports** on the target and lists their current access flags.
* **Lab Discoveries:** Completed in 2.69 seconds. Found **997 closed ports** (returned standard reset markers) and **3 open ports**:

---
### 📸 SCREENSHOT POINT 2: ATTACK SURFACE PORT AUDIT
![Basic Port Scan Output](screenshots/2_basic_port_scan.png)
*Figure 2: Verifying open channel attack surfaces across the official test target environment.*
---

| Port / Protocol | State | Default Service Type |
| :--- | :--- | :--- |
| `22/tcp` | open | ssh |
| `80/tcp` | open | http |
| `9929/tcp` | open | nping-echo |

* **💥 Attacker View:** Identifies possible entry vectors. Port 22 invites brute-force SSH attacks, while Port 80 exposes web applications for vulnerability checks.
* **🛡️ Defender View:** Perimeter compliance checking. Verifies that only designated network avenues are accessible from external routes.

---

### 🧬 Command 3: Service & Version Detection
```bash
nmap -sV scanme.nmap.org
```
* **Mechanics:** Connects directly to open ports and deploys banner queries to extract explicit software names and patch variations.
* **Lab Discoveries:** The execution time lengthened to 10.32 seconds. It returned exact version fingerprints:

---
### 📸 SCREENSHOT POINT 3: SERVICE SOFTWARE BANNER GRABBING
![Service Version Output](screenshots/3_version_detection.png)
*Figure 3: Uncovering application names and patch levels for exploit cross-referencing.*
---

| Port / Protocol | Active Service | Verified Software Version |
| :--- | :--- | :--- |
| `22/tcp` | tcpwrapped | *None identified (connection closed immediately)* |
| `80/tcp` | http | Apache httpd 2.4.7 (Ubuntu) |
| `9929/tcp` | nping-echo | Nping echo |

* **Key Observation:** Port 22 changed to `tcpwrapped`, meaning a protective control cut the session immediately after the handshake. More importantly, the web service exposed an outdated **Apache 2.4.7** application.
* **💥 Attacker View:** Maps out clear exploits by comparing explicit software version tags against public CVE databases.
* **🛡️ Defender View:** Spots outdated internal dependencies so administrators can patch them before adversaries weaponize an attack.

---

### 🐧 Command 4: Operating System Fingerprinting
```bash
sudo nmap -O scanme.nmap.org
```
* **Mechanics:** Requires `sudo` admin permissions to craft custom raw packet headers, matching the system responses against Nmap's local fingerprint database.
* **Lab Discoveries:** Completed in 4.93 seconds with an estimated guess range:

---
### 📸 SCREENSHOT POINT 4: OPERATING SYSTEM FINGERPRINTING
![OS Detection Output](screenshots/4_os_fingerprint.png)
*Figure 4: Inspecting raw network stack anomalies to predict system kernels.*
---

* **Device Type:** General Purpose
* **Running:** Linux 4.X
* **OS Details:** Linux 4.19 to 5.15 (21 network routing hops away)

* **Key Observation:** Fingerprinting is probabilistic. The newer Linux kernel estimate vs. the older Apache application shows that security professionals should cross-verify multiple sources rather than relying entirely on a single tool's guess.
* **💥 Attacker View:** Selects operating-system-specific shellcode payloads (e.g., dropping Linux binaries instead of Windows scripts).
* **🛡️ Defender View:** This scanning technique leaves distinct signatures that alert intrusion detection systems.

---

## 🔧 Troubleshooting Log

### ❌ Problem 1: Scanning the Incorrect Subnet
* **Symptom:** Executing `nmap -sn 172.16.122.0/24` caused the terminal application to hang indefinitely.
* **Diagnosis:** Incorrect range constraint. Nmap wasted time waiting for timeouts across 256 addresses that did not live on my interface.

---
### 📸 SCREENSHOT POINT 5: MISCONFIGURED SUBNET ROUTING
![Wrong Subnet Error](screenshots/error_wrong_subnet.png)
*Figure 5: High latency delay screens caused by entering out-of-scope targets.*
---

* **Fix:** I ran `ifconfig` to verify my interface settings, changing the instruction parameter back to `192.168.1.0/24`.
* **Lesson:** Always confirm local configurations first to verify your active scope.

### ❌ Problem 2: Parameter Verification Fault
* **Symptom:** The console output dropped a fatal warning: `unrecognized option '-0'`.
* **Diagnosis:** A typographical error where I entered the numerical integer `-0` instead of the capital character `-O`.

---
### 📸 SCREENSHOT POINT 6: SYSTEM OPTION PARSING FAULT
![Syntax Typo Error](screenshots/error_syntax_typo.png)
*Figure 6: Command prompt rejection indicating case-sensitive option faults.*
---

* **Fix:** Re-entered the configuration using `sudo nmap -O scanme.nmap.org`.
* **Lesson:** Nmap commands are case-sensitive, and a numerical zero closely resembles an alpha 'O' in most code fonts.

---

## 🗺️ Mapping to MITRE ATT&CK® Framework

| Lab Reconnaissance Target Style | Corresponding MITRE ATT&CK Technical Classification |
| :--- | :--- |
| **Host Discovery Sweep (`-sn`)** | • **T1018** – Remote System Discovery<br>• **T1595.001** – Active Scanning: Scanning IP Blocks |
| **Basic Port Identification** | • **T1046** – Network Service Discovery<br>• **T1595** – Active Scanning |
