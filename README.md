# Nmap-Scanning

import os
import shutil
import subprocess

def create_repository_structure():
    """Generates the clean portfolio directory structure."""
    print("[*] Creating repository directory layout...")
    os.makedirs("screenshots", exist_ok=True)
    print("[+] Successfully initialized 'screenshots/' directory.")

def write_readme():
    """Generates the absolute complete markdown project mapping out explicit screenshot points."""
    print("[*] Compiling README.md documentation file...")
    
    readme_content = """# 🛡️ Foundational Network Reconnaissance with Nmap 7.99

A practical study of foundational network reconnaissance techniques using **Nmap 7.99** on a **Kali Linux VM**. This project explores host discovery, port scanning, service versioning, and OS fingerprinting from both offensive and defensive postures.

* **Learning with:** AndrewCyberJones and Empirical Training
* **Lab Engine:** Nmap 7.99 on Kali Linux (hosted on macOS)

---

## 📋 Project Overview
This project covers the four core Nmap commands that form the bedrock of network reconnaissance. Each scan was executed, analyzed line-by-line, and mapped to defensive Security Operations Center (SOC) tracking points.

### 🔒 Rules of Engagement
* **Local Scope:** Host discovery ran strictly against my own private home network (`192.168.1.0/24`).
* **Remote Scope:** Remote scans targeted `scanme.nmap.org` (45.33.32.156). 
* *Note: The Nmap project explicitly permits scanning this test machine. Exploits and DoS are prohibited, and traffic was capped within the suggested limit of ~12 scans per day.*

---

## 🖥️ Lab Environment

| Component | Detail |
| :--- | :--- |
| **Scanner** | Nmap 7.99 on Kali Linux VM |
| **VM Network Interface** | `eth0` at `192.168.1.126` (Netmask: `255.255.255.0` / `/24`) |
| **Local Target Subnet** | `192.168.1.0/24` (256 addresses) |
| **Remote Target Host** | `scanme.nmap.org` |

---

## 🛠️ Execution Walkthrough

### ⚙️ Phase 0: Network Mapping
Before interacting with the network, running `ifconfig` identified the active local range:
* **IP / Mask:** `192.168.1.126` with netmask `255.255.255.0`
* **Resulting Range:** `192.168.1.0/24` (covers `.0` to `.255`)

```bash
ifconfig eth0
```

📸 **[SCREENSHOT POINT: Place your terminal window showing the ifconfig output here]**
*File suggestion: screenshots/0_ifconfig_output.png*

---

### 🔍 Command 1: Host Discovery (Ping Sweep)
```bash
nmap -sn 192.168.1.0/24
```
* **Mechanics:** Bypasses port scanning entirely. On local subnets, Nmap leverages raw **ARP requests**. ARP is highly reliable because target hosts must reply to map hardware addresses, even if their host OS firewalls drop ICMP echo requests.
* **Lab Discoveries:** Identified **14 active hosts** in 2.64 seconds. Mapping MAC addresses to vendors exposed the network architecture:

📸 **[SCREENSHOT POINT: Place your terminal output of the nmap -sn command run here]**
*File suggestion: screenshots/1_ping_sweep.png*

| Observed IP | Detected Hardware Vendor / Hostname | Inferred Asset Profile |
| :--- | :--- | :--- |
| `192.168.1.1` | Icotera A/S | ISP Home Gateway / Router |
| *Multiple* | Amazon Technologies | Echo Smart Speakers / IoT |
| *Two Hosts* | Vestel Elektronik | Consumer Smart TVs |
| *Two Hosts* | TP-Link (`Tapo_Bulb` / `L510`) | Smart Automation Lightbulbs |
| *One Host* | Ring | Smart Doorbell / Security Camera |
| *Multiple* | *Unknown* | Mobile devices running MAC randomization |
| `192.168.1.X` | "Mac" (Latency: `0.0002s`) | Virtualization Hypervisor Host (Mac) |
| `192.168.1.126`| *None Listed* | Local Kali Linux Scanning Instance |

* **💥 Attacker View:** Low-cost blueprinting. Quickly identifies soft IoT targets on a network.
* **🛡️ Defender View:** Core asset inventory tracking. SOC monitors flag single sources querying whole subnets via sequential ARP/ICMP over brief windows.

---

### 🚪 Command 2: Basic Port Scan
```bash
nmap scanme.nmap.org
```
* **Mechanics:** Evaluates the **1,000 most common TCP ports** by default.
* **Lab Discoveries:** Finished in 2.69 seconds. Found **997 closed ports** (returned TCP RST flags) and **3 open ports**:

📸 **[SCREENSHOT POINT: Place your basic top-1000 nmap scan output window here]**
*File suggestion: screenshots/2_basic_port_scan.png*

| Port / Protocol | State | Default Service Profile |
| :--- | :--- | :--- |
| `22/tcp` | open | ssh |
| `80/tcp` | open | http |
| `9929/tcp` | open | nping-echo |

* **💥 Attacker View:** Identifies exposed entry channels. SSH (22) indicates a brute-force route; HTTP (80) points to a web server surface.
* **🛡️ Defender View:** Surface reduction auditing. Used to verify that firewall barriers are actively filtering unauthorized services.

---

### 🧬 Command 3: Service & Version Detection
```bash
nmap -sV scanme.nmap.org
```
* **Mechanics:** Probes open TCP connections by sending application-layer challenges to verify software banners and version signatures.
* **Lab Discoveries:** Execution extended to 10.32 seconds. Exposed deep environment information:

📸 **[SCREENSHOT POINT: Place your detailed nmap -sV service detection console output here]**
*File suggestion: screenshots/3_version_detection.png*

| Port / Protocol | Active Service | Verified Software Version |
| :--- | :--- | :--- |
| `22/tcp` | tcpwrapped | *None identified (connection reset immediately)* |
| `80/tcp` | http | Apache httpd 2.4.7 (Ubuntu) |
| `9929/tcp` | nping-echo | Nping echo |

* **Key Takeaway:** Port 22 shifted to `tcpwrapped`, signaling that TCP Wrappers or an ACL closed the socket instantly upon handshake completion. The discovery of an older **Apache 2.4.7** web server provides an exact tag for CVE vulnerability cross-referencing.
* **💥 Attacker View:** Transitions general observations into precise attack paths via version exploitation lookup.
* **🛡️ Defender View:** Finds end-of-life or unpatched assets before adversarial discovery.

---

### 🐧 Command 4: Operating System Fingerprinting
```bash
sudo nmap -O scanme.nmap.org
```
* **Mechanics:** Requires `sudo` (root privileges) to construct and inspect abnormal raw packet headers. Evaluates TCP/IP stack performance quirks against Nmap's OS fingerprint signature library.
* **Lab Discoveries:** Code resolved in 4.93 seconds with a probabilistic judgment:

📸 **[SCREENSHOT POINT: Place your sudo nmap -O operating system fingerprint terminal output here]**
*File suggestion: screenshots/4_os_fingerprint.png*

* **Device Type:** General Purpose
* **OS Generation:** Linux 4.X
* **Estimated Fingerprint Range:** Linux 4.19 to 5.15 (21 Network Hops away)

* **Key Takeaway:** Fingerprinting is probabilistic. The newer Linux Kernel estimation vs. the older Apache 2.4.7 package reminds analysts to synthesize and cross-verify facts rather than trust a single data vector.
* **💥 Attacker View:** Directs payload curation (e.g., selecting Linux-specific binaries over Windows exploits).
* **🛡️ Defender View:** Abnormal packet headers generated by OS scans spark high-fidelity IDS alerts.

---

## 🔧 Troubleshooting Log

### ❌ Problem 1: Scanning the Incorrect Range
* **Symptom:** Run command `nmap -sn 172.16.122.0/24` hung endlessly without returning discoveries.
* **Diagnosis:** Incorrect subnet selection. Nmap spent its window waiting for timeouts on addresses that did not map to the physical interface context.

📸 **[SCREENSHOT POINT: Place your terminal showing the hung/blank scanning output or timeout delay here]**
*File suggestion: screenshots/error_wrong_subnet.png*

* **Mitigation:** Verified active properties with `ifconfig`, changing the input target range to `192.168.1.0/24`.
* **Takeaway:** Always confirm active constraints using local system tools to preserve scoping rules.

### ❌ Problem 2: Syntactical Typo on OS Flag
* **Symptom:** Shell dropped error: `unrecognized option '-0'`.
* **Diagnosis:** Typo replacement of capital alpha character `-O` with numerical integer `-0`.

📸 **[SCREENSHOT POINT: Place your command prompt error indicating 'unrecognized option -0' here]**
*File suggestion: screenshots/error_syntax_typo.png*

* **Mitigation:** Adjusted command string to read `sudo nmap -O <target>`.

---

## 🗺️ Mapping to MITRE ATT&CK®

| Scan Style | MITRE ATT&CK Reconnaissance & Discovery Mapping |
| :--- | :--- |
| **Host Discovery Sweep (`-sn`)** | • **T1018** – Remote System Discovery<br>• **T1595.001** – Active Scanning: Scanning IP Blocks |
| **Port Mapping Scan** | • **T1046** – Network Service Discovery<br>• **T1595** – Active Scanning |
| **Service Version Detection (`-sV`)** | • **T1592.002** – Gather Victim Host Information: Software<br>• **T1046** – Network Service Discovery |
| **OS Fingerprinting (`-O`)** | • **T1592** – Gather Victim Host Information (Configuration / OS) |

---

## 🛡️ Detection & Incident Response Strategy
1. **Network Layer Footprints:** Watch for high volumes of localized ARP or ICMP requests stemming from a single internal address.
2. **Transport Layer Anomalies:** Track high-frequency connection rejections or drop drops from a centralized IP across diverse port paths.
3. **Application & Stack Integrity:** Monitor for raw, malformed packets and sudden banner probing alerts flagged via tools like **Suricata** or **Zeek**.
4. **Playbook Action:** Correlate the source IP against authorized internal vulnerability scanner assets. If unauthorized, immediately restrict or isolate the source host via the EDR framework or Network Access Control.
"""
    with open("README.md", "w", encoding="utf-8") as f:
        f.write(readme_content.strip())
    print("[+] Successfully wrote README.md file.")

def write_scanner():
    """Generates the zero-dependency functional python logging tool."""

