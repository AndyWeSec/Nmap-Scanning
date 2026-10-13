# Nmap-Scanning
This project covers the four Nmap commands that form the foundation of network reconnaissance.

#!/usr/bin/env python3
"""
Nmap Lab Workflow Automation Tool
Wraps four foundational scanning phases: Ping Sweeps, Port Scans, Version Tracking, and OS Identification.
"""

import os
import sys

try:
    import nmap
except ImportError:
    print("[-] Error: python-nmap dependency missing.")
    print("[*] Install it via: pip install python-nmap")
    sys.exit(1)

def ensure_root():
    """Ensures raw packet manipulation options have necessary system access."""
    if os.name != 'nt' and os.geteuid() != 0:
        print("[-] Error: Root privileges required for advanced Nmap features (like OS Fingerprinting).")
        print("[*] Re-run command using: sudo python3 scanner.py")
        sys.exit(1)

def display_menu():
    print("\n" + "="*50)
    print(" 🛡️  FOUNDATIONAL NMAP SCANNING LAB TOOL")
    print("="*50)
    print("1. Local Host Discovery Sweep     [-sn] (Target your /24 Network)")
    print("2. Basic Top-1000 Port scan       [Default TCP]")
    print("3. Service & Version Detection    [-sV]")
    print("4. Operating System Fingerprint   [-O] (Requires Root)")
    print("5. Exit Laboratory")
    print("="*50)

def execute_scan():
    nm = nmap.PortScanner()
    
    while True:
        display_menu()
        choice = input("[+] Select Lab Exercise (1-5): ").strip()
        
        if choice == '5':
            print("[*] Exiting Lab Environment. Safe Scanning!")
            break
            
        if choice not in ['1', '2', '3', '4']:
            print("[-] Invalid choice selection.")
            continue
            
        target = input("[+] Define Target Host, Subnet, or Domain: ").strip()
        if not target:
            print("[-] Error: Target identifier cannot be blank.")
            continue

        # Prevent Problem 1 (Incorrect Subnet Range awareness)
        if choice == '1' and not target.endswith('/24'):
            print("[!] Warning: Host discovery sweeps typically require CIDR ranges (e.g., /24).")
            confirm = input(f"[?] Proceed with scanning target '{target}' exactly? (y/n): ")
            if confirm.lower() != 'y':
                continue

        try:
            if choice == '1':
                print(f"[*] Dispatching Ping Sweep (-sn) over {target}...")
                nm.scan(hosts=target, arguments='-sn')
            elif choice == '2':
                print(f"[*] Mapping Top 1000 standard ports on {target}...")
                nm.scan(hosts=target, arguments='')
            elif choice == '3':
                print(f"[*] Initiating application service & banner probing (-sV) against {target}...")
                nm.scan(hosts=target, arguments='-sV')
            elif choice == '4':
                ensure_root()
                # Prevent Problem 2 (Enforces -O flag instead of accidental -0 integer)
                print(f"[*] Issuing raw stack OS fingerprint evaluation (-O) against {target}...")
                nm.scan(hosts=target, arguments='-O')

            # Render parsed discoveries
            for host in nm.all_hosts():
                print("\n" + "#"*45)
                print(f"🎯 Target Host: {host} | State: {nm[host].state()}")
                print("#"*45)

                if 'osmatch' in nm[host] and nm[host]['osmatch']:
                    print("\n[🧬 OS Fingerprint Matches]")
                    for match in nm[host]['osmatch']:
                        print(f" -> {match['name']} ({match['accuracy']}% confidence)")

                for proto in nm[host].all_protocols():
                    print(f"\n[🔌 Protocol: {proto.upper()}]")
                    ports = sorted(nm[host][proto].keys())
                    
                    print(f"{'PORT':<10}{'STATE':<15}{'SERVICE':<15}{'VERSION VERSION'}")
                    print("-" * 60)
                    for port in ports:
                        state = nm[host][proto][port]['state']
                        name = nm[host][proto][port]['name']
                        product = nm[host][proto][port].get('product', '')
                        version = nm[host][proto][port].get('version', '')
                        version_str = f"{product} {version}".strip()
                        
                        print(f"{f'{port}/{proto}':<10}{state:<15}{name:<15}{version_str if version_str else 'N/A'}")
                        
        except Exception as err:
            print(f"\n[-] Execution Blocked: {err}")
            print("[-] Ensure Nmap binary dependencies are functional on your system pathway.")

if __name__ == '__main__':
    try:
        execute_scan()
    except KeyboardInterrupt:
        print("\n[-] Lab execution stopped by user command.")
        sys.exit(0)
