# 🛡️ Foundational Network Reconnaissance with Nmap

A practical study of foundational network reconnaissance techniques using **Nmap 7.99** on a **Kali Linux VM**. This project explores host discovery, port scanning, service versioning, and OS fingerprinting from both offensive and defensive postures.

* **Learning with:** [AndrewCyberJones](https://www.youtube.com/@andrewcyberjones) and [Empirical Training](https://www.instagram.com/empiricaltraining/)
* **Lab Engine:** Nmap 7.99 on Kali Linux (running in a virtual machine on my Mac)

---

## 📋 Overview
This project covers the four Nmap commands that form the foundation of network reconnaissance: finding live hosts on a network, scanning for open ports, identifying the software and versions behind those ports, and guessing the operating system. I ran each one from a Kali Linux virtual machine, read the output line by line, and then looked at each step from both sides: what an attacker learns, and how a SOC analyst would spot and use the same activity.

The host discovery scan ran only against my own home network. The other three scans ran against scanme.nmap.org, the Nmap project's official test target, which the project explicitly permits people to scan with Nmap.

---

## 🎯 Why this matters to a SOC analyst
* **Recognising reconnaissance:** Nearly every attack starts with scanning. If I know exactly what each Nmap scan looks like from the attacker's side, I know what to look for in firewall, IDS and network logs.
* **Asset visibility:** The same host discovery scan is how defenders build an inventory of what is on their network. You can't protect devices you don't know exist.
* **Reading scan output properly:** Open, closed and filtered ports, service versions and OS guesses all mean something specific. Knowing how to interpret them is what turns raw output into a finding.
* **Interview answer:** To explain how an attacker maps a target, I can walk through these four scans in order: who is there, what is open, what is running on it, and what is it running on.

---

## 🔒 Rules of engagement
* I only scanned my own home network and `scanme.nmap.org`.
* `scanme.nmap.org` is run by the Nmap project as a test target. Scanning it with Nmap is allowed, but exploits and denial-of-service attempts are not, and the project asks people to keep scans to roughly a dozen a day.
* **Never scan a host or network you don't own or have written permission to test.**

---

## 🖥️ Lab environment

| Component | Detail |
| :--- | :--- |
| **Scanner** | Nmap 7.99 on Kali Linux, running in a virtual machine on my Mac |
| **VM network** | `eth0` at `192.168.1.126` with netmask `255.255.255.0` (a `/24` network), on my home network |
| **Local target** | `192.168.1.0/24`, my own home network |
| **Remote target** | `scanme.nmap.org` (45.33.32.156), the Nmap project's official test host |

---

## 🛠️ Walkthrough

### Setup: finding my own network range
Before scanning my network, I had to know its range. Running `ifconfig` showed `eth0` with `inet 192.168.1.126` and `netmask 255.255.255.0`. A `255.255.255.0` mask is a `/24`, so my network is `192.168.1.0/24`, covering 256 addresses (`.0` to `.255`).

```bash
ifconfig
```

<img width="1277" height="820" alt="ifconfig output showing eth0 at 192.168.1.126 with netmask 255.255.255.0" src="https://github.com/user-attachments/assets/dade1276-e664-4777-affc-536b26d18ffa" />

---

### Command 1: Host discovery (ping sweep)
```bash
nmap -sn 192.168.1.0/24
```
* **nmap** is the scanner.
* **-sn** means no port scan. It only checks which hosts are alive.
* **192.168.1.0/24** is the range to check.

**What it does:** It asks every address on the network whether anything is there and lists the ones that answer. On a local network, Nmap uses ARP requests for this, which is why the output includes MAC addresses. ARP is reliable because even devices that ignore ping have to answer it to communicate on the network.

**What I found:** Nmap done: 256 IP addresses (14 hosts up) scanned in 2.64 seconds. The output showed each host's IP, latency, and the vendor name looked up from its MAC address. Reading the vendors told me a lot about the network without touching a single port:

<img width="1277" height="820" alt="Host discovery results with MAC addresses blurred" src="https://github.com/user-attachments/assets/8dc07624-4c48-4c2e-be45-49a4ce327d26" />

<!-- Add the second sweep screenshot (4.png, the end of the results showing 14 hosts up) here. Use the version with the MAC addresses blurred. -->

| What Nmap showed | Likely meaning |
| :--- | :--- |
| `192.168.1.1`, vendor Icotera A/S | My router (the vendor makes ISP routers and .1 is the usual gateway address) |
| Several Amazon Technologies devices | Amazon smart speakers or similar devices |
| Two Vestel Elektronik devices | Probably smart TVs |
| Two TP-Link devices (hostnames `Tapo_Bulb` and `L510`) | Smart bulbs |
| One Ring device | A doorbell or camera |
| Devices with vendor "Unknown" | Probably phones or laptops using MAC address randomisation, where the address is private and doesn't match a vendor |
| Host "Mac" at 0.0002s latency | My own Mac, which hosts the Kali VM (the tiny latency gives it away) |
| Host "kali" with no MAC shown | The VM I scanned from. Nmap doesn't print a MAC for the machine it runs on. |

* **Attacker view:** This is the first step of reconnaissance. A sweep produces a target list in seconds, and smart-home and IoT devices are often the weakest on a network.
* **Defender view:** This is asset inventory. A defender runs the same scan to spot unknown or rogue devices. In a SOC, one host sending ARP or ICMP requests to every address in a subnet within a few seconds is a classic scanning signature, and tools like Zeek and Suricata can flag it.

---

### Command 2: Basic port scan
```bash
nmap scanme.nmap.org
```
**What it does:** With no flags, Nmap scans the 1,000 most common TCP ports on the target and reports each one's state: open (a service is listening), closed (the host answered that nothing is listening) or filtered (a firewall is hiding the answer).

**What I found:** The host was up with 0.22s latency. Of the 1,000 ports, 997 were closed (the scan reported them as "reset") and three were open:

<img width="1277" height="820" alt="Basic Nmap port scan of scanme.nmap.org showing ports 22, 80 and 9929 open" src="https://github.com/user-attachments/assets/88be2809-c818-42d2-ad3b-6574f7cd3dfe" />

| Port | State | Service |
| :--- | :--- | :--- |
| **22/tcp** | open | ssh |
| **80/tcp** | open | http |
| **9929/tcp** | open | nping-echo |

Nmap also listed another address for the host (an IPv6 address) that it did not scan. The scan finished in 2.69 seconds.

* **Attacker view:** Every open port is a possible way in. SSH on 22 is a remote login service that attackers try to brute-force, and HTTP on 80 is a web server that may contain vulnerable applications.
* **Defender view:** The open ports are the attack surface. A defender runs this same scan against their own servers to confirm only the intended ports are open, and a rapid series of connection attempts to many different ports from one source is the signature of a port scan in firewall logs.

---

### Command 3: Service and version detection
```bash
nmap -sV scanme.nmap.org
```
* **-sV** makes Nmap connect to each open port and work out what software is running and which version.

**What I found:** This scan took 10.32 seconds, longer than the basic scan, because it probes each service. The results were more detailed:

<img width="1277" height="820" alt="Nmap -sV output showing Apache httpd 2.4.7 on port 80 and tcpwrapped on port 22" src="https://github.com/user-attachments/assets/a16408e8-cb7b-402e-999a-92b163f986d8" />

| Port | Service | Version |
| :--- | :--- | :--- |
| **22/tcp** | tcpwrapped | (none identified) |
| **80/tcp** | http | Apache httpd 2.4.7 (Ubuntu) |
| **9929/tcp** | nping-echo | Nping echo |

Two things stood out. First, port 22 changed from "ssh" in the basic scan to "tcpwrapped". That means the connection was accepted and then closed straight away, which is typical when access control rules such as TCP wrappers refuse the connection, so Nmap couldn't identify the service. Second, the web server was Apache 2.4.7 on Ubuntu, a very old release. In a real assessment, an exact version number is what you search against CVE databases to find known vulnerabilities.

* **Attacker view:** Version numbers turn "there's a web server" into "there's a web server with these known flaws", which tells the attacker which exploit to try.
* **Defender view:** Version scanning is how defenders find outdated software before attackers do. It also shows what your own servers reveal about themselves, and hiding version banners makes an attacker's job harder.

---

### Command 4: Operating system detection
```bash
sudo nmap -O scanme.nmap.org
```
* **-O** (a capital letter O) turns on OS detection.
* **sudo** is needed because OS detection sends specially crafted raw packets, which requires root privileges.

**What it does:** Nmap sends a series of unusual probes and compares how the target responds with a database of known operating system fingerprints.

**What I found:** The scan finished in 4.93 seconds, with the same three open ports. The OS results were:

<img width="1277" height="820" alt="Nmap -O output showing Linux 4.19 to 5.15 and 21 hops, with the earlier -0 typo error above it" src="https://github.com/user-attachments/assets/cef8bf7b-7eab-4f03-a9cf-74c276a3d70f" />

* **Device type:** general purpose
* **Running:** Linux 4.X
* **OS details:** Linux 4.19 to 5.15
* **Network distance:** 21 hops

The result is a range and a best guess, not a certainty. The Linux 4.19 to 5.15 guess is also notably newer than the very old Apache version found in Command 3, a reminder that fingerprinting is probabilistic and that results should be cross-checked rather than trusted blindly.

* **Attacker view:** Knowing the operating system lets an attacker choose OS-specific exploits and tools.
* **Defender view:** OS detection involves unusual packet patterns that intrusion detection systems commonly flag, and defenders can use it to check that machines are running the operating systems they should be.

---

## 🔧 Troubleshooting log

### Problem 1: Scanning the wrong range
* **Symptom:** My first host discovery scan, `nmap -sn 172.16.122.0/24`, sat there with no results.
* **Diagnosis:** That range wasn't the network my interface was actually on. Nmap had to wait for timeouts across 256 addresses that never answered.
* **Fix:** Take the range from `ifconfig`. My interface was `192.168.1.126` with a `/24` mask, so the right range was `192.168.1.0/24`. (I also ran it as `192.168.1.255/24`, which works because Nmap applies the mask, but `192.168.1.0/24` is the clean form.)
* **Lesson:** Always confirm your own range before scanning, which also keeps you inside your scope.

<img width="1277" height="820" alt="The first attempt scanning 172.16.122.0/24 with no results" src="https://github.com/user-attachments/assets/764335af-ec96-425f-b35a-2887a8be1d68" />

### Problem 2: A typo in the OS detection flag
* **Symptom:** `unrecognized option '-0'` followed by a prompt to see `nmap -h`.
* **Diagnosis:** I had typed the number zero (`-0`) instead of the capital letter O (`-O`).
* **Fix:** Re-run with `sudo nmap -O scanme.nmap.org`.
* **Lesson:** Nmap flags are case-sensitive and a zero looks very like an O in a terminal font. The error output points you to `nmap -h` for the correct options. (The error is visible at the top of the Command 4 screenshot.)

---

## 🗺️ Mapping to MITRE ATT&CK
These four scans are standard reconnaissance and discovery activity. The closest matches in the ATT&CK framework are below (check the exact wording on attack.mitre.org before quoting them).

| Scan | Technique |
| :--- | :--- |
| Host discovery sweep (`-sn`) | T1018 Remote System Discovery, and T1595.001 Active Scanning: Scanning IP Blocks |
| Port scan | T1046 Network Service Discovery, and T1595 Active Scanning |
| Service and version detection (`-sV`) | T1592.002 Gather Victim Host Information: Software, and T1046 Network Service Discovery |
| OS detection (`-O`) | T1592 Gather Victim Host Information (operating system and client configuration details) |

---

## 🛡️ How a SOC would detect this
* **Host sweep:** One source sending ARP or ICMP requests to many addresses in a short time.
* **Port scan:** One source making connection attempts to many ports on one host, producing lots of refused connections or firewall drops.
* **Version and OS scans:** Unusual or malformed packet patterns and repeated probing of the same open services, which IDS tools such as Suricata and Zeek commonly alert on.
* **Response:** Check whether the source is an authorised scanner or vulnerability tool first. If it isn't, investigate the source device and block or isolate it.

---

## 🧠 Skills demonstrated
* Running and reading Nmap host discovery, port, service version and OS detection scans
* Working out a network range from interface details
* Interpreting open, closed and filtered ports, "tcpwrapped" results and OS fingerprint ranges
* Thinking about each scan from both the attacker's and the defender's side
* Staying within scope and following the rules of engagement
* Troubleshooting scan problems from error output
* Mapping scanning activity to MITRE ATT&CK

---

## 🔭 Next steps
* Capture my own scans in Wireshark to see exactly what a sweep and a port scan look like on the wire, then write detection rules for them.
* Try the Nmap scripting engine (`--script`) against my own devices.
* Compare the OS and version results against other sources of information about the target.

---

*Scans were run only against my own network and the Nmap project's official test host, `scanme.nmap.org`.*

