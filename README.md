# Nmap-Scanning
This project covers the four Nmap commands that form the foundation of network reconnaissance.
# 🛡️ Foundational Network Reconnaissance with Nmap 7.99

A practical study of foundational network reconnaissance techniques using **Nmap 7.99** on a **Kali Linux VM**. This project explores host discovery, port scanning, service versioning, and OS fingerprinting from both offensive and defensive postures.

* **Learning with:** AndrewCyberJones and Empirical Training
* **Lab Engine:** Nmap 7.99 on Kali Linux (hosted on macOS)

---

## 📋 Project Overview
This project covers the four core Nmap commands that form the bedrock of network reconnaissance. Each scan was executed, analyzed line-by-line, and mapped to defensive Security Operations Center (SOC) tracking points.

### 🔒 Rules of Engagement
* **Local Scope:** Host discovery ran strictly against my own private home network (`192.168.1.0/24`).
* **Remote Scope:** Remote scans targeted `scanme.nmap.org` (45.33.32.156). 

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
Before interacting with the network, running `ifconfig` identified the active local range.

![Ifconfig Network Range Diagnostics](screenshots/ifconfig_range.png)
*Figure 0: Locating the local interface parameters via terminal mapping utilities.*

---

### 🔍 Command 1: Host Discovery (Ping Sweep)
```bash
nmap -sn 192.168.1.0/24
```
* **Mechanics:** Bypasses port scanning entirely. On local subnets, Nmap leverages raw **ARP requests**. ARP is highly reliable because target hosts must reply to map hardware addresses, even if their host OS firewalls drop ICMP echo requests.

![Host Discovery Sweep Terminal Execution](screenshots/command1_ping_sweep.png)
*Figure 1: Terminal verification of live hosts discovered within the local address space.*

| Observed IP | Detected Hardware Vendor / Hostname | Inferred Asset Profile |
| :--- | :--- | :--- |
| `192.168.1.1` | Icotera A/S | ISP Home Gateway / Router |
| *Multiple* | Amazon Technologies | Echo Smart Speakers / IoT |
| *Two Hosts* | Vestel Elektronik | Consumer Smart TVs |
| *Two Hosts* | TP-Link (`Tapo_Bulb` / `L510`) | Smart Automation Lightbulbs |
| `192.168.1.X` | "Mac" (Latency: `0.0002s`) | Virtualization Hypervisor Host (Mac) |

---

### 🚪 Command 2: Basic Port Scan
```bash
nmap scanme.nmap.org
```
* **Mechanics:** Evaluates the **1,000 most common TCP ports** by default.

![Basic Port Mapping Script Run](screenshots/command2_port_scan.png)
*Figure 2: Active connection footprints showing raw TCP service accessibility metrics.*

| Port / Protocol | State | Default Service Profile |
| :--- | :--- | :--- |
| `22/tcp` | open | ssh |
| `80/tcp` | open | http |
| `9929/tcp` | open | nping-echo |

---

### 🧬 Command 3: Service & Version Detection
```bash
nmap -sV scanme.nmap.org
```
* **Mechanics:** Probes open TCP connections by sending application-layer challenges to verify software banners and version signatures.

![Service Software Version Diagnostics](screenshots/command3_version_detection.png)
*Figure 3: Deep probing output identifying the specific software distributions listening behind available sockets.*

| Port / Protocol | Active Service | Verified Software Version |
| :--- | :--- | :--- |
| `22/tcp` | tcpwrapped | *None identified (connection reset immediately)* |
| `80/tcp` | http | Apache httpd 2.4.7 (Ubuntu) |

---

### 🐧 Command 4: Operating System Fingerprinting
```bash
sudo nmap -O scanme.nmap.org
```
* **Mechanics:** Requires `sudo` (root privileges) to construct and inspect abnormal raw packet headers. Evaluates TCP/IP stack performance quirks against Nmap's OS fingerprint signature library.

![Operating System Stack Fingerprinting](screenshots/command4_os_detection.png)
*Figure 4: Algorithmic network responses generating probabilistic kernel guesses.*

---

## 🔧 Troubleshooting Log

### ❌ Problem 1: Scanning the Incorrect Range
* **Symptom:** Run command `nmap -sn 172.16.122.0/24` hung endlessly without returning discoveries.

![Timeout Errors from Out of Scope Subnets](screenshots/trouble_wrong_subnet.png)
*Figure 5: High latency timeouts encountered when executing requests outside active interface routes.*

### ❌ Problem 2: Syntactical Typo on OS Flag
* **Symptom:** Shell dropped error: `unrecognized option '-0'`.

![Syntax Parameter Warnings](screenshots/trouble_syntax_typo.png)
*Figure 6: System console output flagging case-sensitive validation faults.*


---

## 🔭 Next Milestones
* [ ] Capture a ping sweep and port scan inside **Wireshark** to evaluate packet footprints on the wire.
* [ ] Write custom detection alerts for active scanning tools within an IDS environment.
* [ ] Test the Nmap Scripting Engine (`--script`) flags against owned sandbox devices.

#!/usr/bin/env python3
"""
Nmap Lab Workflow Automation Tool - Native System Bridge & Logging Edition
Runs raw scans safely and saves output logs automatically to help your troubleshooting layout.
"""

import os
import sys
import subprocess
import shutil

def check_environment():
    """Verify that Nmap is installed and available in the system PATH."""
    if not shutil.which("nmap"):
        print("[-] Error: The 'nmap' binary was not found in your system's path.")
        print("[*] Install it on Kali/Ubuntu with: sudo apt install nmap")
        sys.exit(1)

def ensure_root():
    """Ensures raw packet manipulation options have root access permissions."""
    if os.name != 'nt' and os.geteuid() != 0:
        print("\n[-] Error: Root privileges required for raw packet analysis (like OS Fingerprinting).")
        print("[*] Re-run command using: sudo python3 scanner.py")
        sys.exit(1)

def display_menu():
    print("\n" + "="*60)
    print(" 🛡️  FOUNDATIONAL NMAP LAB LOGGER TOOL")
    print("="*60)
    print("1. Local Host Discovery Sweep     [-sn]")
    print("2. Basic Top-1000 Port Scan       [Default TCP]")
    print("3. Service & Version Detection    [-sV]")
    print("4. Operating System Fingerprint   [-O] (Requires Root)")
    print("5. Exit Laboratory")
    print("="*60)

def execute_scan():
    check_environment()
    log_file = "nmap_scan_output.txt"
    
    while True:
        display_menu()
        choice = input("[+] Select Lab Exercise (1-5): ").strip()
        
        if choice == '5':
            print("[*] Exiting Lab Environment. Logs saved to: " + log_file)
            break
            
        if choice not in ['1', '2', '3', '4']:
            print("[-] Invalid choice selection.")
            continue
            
        target = input("[+] Define Target Host, Subnet, or Domain: ").strip()
        if not target:
            print("[-] Error: Target identifier cannot be blank.")
            continue

        if choice == '1':
            cmd = ["nmap", "-sn", target]
        elif choice == '2':
            cmd = ["nmap", target]
        elif choice == '3':
            cmd = ["nmap", "-sV", target]
        elif choice == '4':
            ensure_root()
            cmd = ["nmap", "-O", target]

        try:
            print(f"\n[*] Executing and logging: {' '.join(cmd)}")
            print("-" * 60)
            
            # Execute command tracking real-time line streams directly out to the shell terminal interface
            process = subprocess.Popen(cmd, stdout=subprocess.PIPE, stderr=subprocess.STDOUT, text=True)
            
            with open(log_file, "a") as f:
                f.write(f"\n\n--- Executing Scan Command: {' '.join(cmd)} ---\n")
                while True:
                    output = process.stdout.readline()
                    if output == '' and process.poll() is not None:
                        break
                    if output:
                        clean_output = output.strip()
                        print(clean_output)
                        f.write(clean_output + "\n")
            
            print("-" * 60)
            print(f"[+] Output captured and logged successfully into '{log_file}'.")
            print("[!] Tip: Take a desktop screenshot of your window now to pair with this console output!")
                        
        except Exception as err:
            print(f"\n[-] Execution Error: {err}")

if __name__ == '__main__':
    try:
        execute_scan()
    except KeyboardInterrupt:
        print("\n\n[-] Lab execution stopped by user command.")
        sys.exit(0)
