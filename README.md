# 🛡️ Enterprise Cybersecurity Engineering Home Lab

## 📑 Engineering Journal

---

### 🗺️ Master Architecture Blueprint

* 🟢 Phase 1: Hardening — COMPLETED
* 🟢 Phase 2: Active IPS — COMPLETED
* 🟢 Phase 3: Recon & Perimeter Audit (Nmap) — COMPLETED (Validated)
* 🟢 Phase 4: Network Packet Forensics (Wireshark + tcpdump) — COMPLETED & VALIDATED
* 🔵 Phase 5: SIEM (Splunk) — PLANNED

---

### 💻 Phase 1: Local Linux Server Deployment and Baseline Hardening

* 📊 Status: 100% COMPLETE & VERIFIED
* 🖥️ Host Machine: JayPC (AMD Ryzen 3 3200G, 16GB RAM)
* ⚙️ Hypervisor: Oracle VirtualBox
* 🐧 Guest OS: Ubuntu Server 24.04 LTS (Headless Setup)
* 📌 Dedicated Management IP: 192.168.1.200

#### 🎯 Project Milestones Achieved

* Hardware Virtualization Configuration: Enabled SVM Mode in the AMD BIOS to unlock hypervisor capabilities on the host system.
* Resource Allocation: Provisioned a balanced, headless guest instance utilizing 2 vCPU Cores and 2GB RAM.
* Layer 2 Networking Architecture: Configured the VirtualBox network interface card to Bridged Mode to seamlessly attach the server to the physical home subnet.
* Secure Access Implementation: Installed and hardened OpenSSH Server components to handle administrative connectivity.
* Zero-Trust Administrative Management: Established encrypted, concurrent SSH terminal sessions from specialized client machines (Windows PC and MacBook Air).
* Perimeter Hardening: Deployed Uncomplicated Firewall (UFW) enforcing a default-deny inbound posture.
* Network Whitelisting: Migrated the perimeter access rule from a globally exposed network configuration to a targeted local subnet mask whitelist (192.168.1.0/24), neutralizing unauthorized access vectors.
* Static IP Orchestration: Overrode the dynamic host configuration protocol (DHCP) by injecting a permanent static routing table profile via Netplan configuration.

---

### 🚨 Phase 2: Manual Log Analysis and Active Intrusion Prevention (IPS)

* 📊 Status: 100% COMPLETE & VERIFIED
* 🚀 Core Software Engine: Nginx Web Server Ecosystem
* 🛡️ IPS Daemon: Fail2Ban System Automation

#### 🎯 Project Milestones Achieved

* Forensic Log Parsing Infrastructure: Developed multi-column parsing pipelines using Linux data manipulation tools to inspect real-time application access telemetry.
* Pattern Discovery: Isolated automated directory discovery brute‑force signatures by mapping network communication behaviors to specific internal server resources.
* Automated Threat Suppression: Implemented a custom regular expression filter linked to a dynamic Fail2Ban jail policy to automatically manipulate the host firewall state upon threat detection.
* Perimeter Policy Hardening: Configured strict, unnegotiated packet dropping mechanisms to fully neutralize active network scanners.

#### 🔍 Deep-Dive: Phase 2 Incident Analysis and Telemetry

##### 1. Forensic Log Parsing (The Command‑Line Pipeline)

To isolate unauthorized scanning behavior, specific text manipulation pipelines were executed directly against the live Nginx access log (/var/log/nginx/access.log):

* 📍 Source Isolation (Tracking Hit Volume per Host):
  sudo awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr

* 🗺️ URI Intent Analysis (Tracking Targeted Directory Paths):
  sudo awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -nr

* 🔄 The Cross‑Reference Matrix (Tying Unique Hosts to Action Profiles):
  sudo awk '{print $1, $7}' /var/log/nginx/access.log | sort | uniq -c | sort -nr

##### 📈 Analytical Findings

The output explicitly mapped out two distinct operational profiles across the local network fabric:

1. 192.168.1.41 (Mobile Endpoint): Valid user behavior interacting exclusively with the site root directory (/).
2. 192.168.1.2 (Windows Endpoint): Aggressive, high‑velocity directory traversal scanner hunting for administrative entry points (/wp‑login.php, /secret‑passwords.txt, /admin), generating a continuous series of HTTP 404 Not Found errors.

##### 2. Automated Intrusion Prevention System (IPS) Configuration

To programmatically block the malicious profile, an automated active response loop was deployed:

* 📑 The Signature Filter (/etc/fail2ban/filter.d/nginx-404.conf):
  [Definition]
  failregex = ^<HOST> - - \[.*\] ".*" 404 .*

* ⚙️ The Enforcement Jail Configuration (/etc/fail2ban/jail.d/nginx-404.local):
  Configured a strict operational policy: if an IP triggers 5 failed resource requests (404) inside a rolling 10‑second window, Fail2Ban dynamically modifies the kernel firewall layer to inject a hard network block for a 10‑minute duration.

  [nginx-404]
  enabled  = true
  port     = http,https
  filter   = nginx-404
  logpath  = /var/log/nginx/access.log
  backend  = auto
  maxretry = 5
  findtime = 10
  bantime  = 600
  action   = ufw[action=deny]

---

### 📡 Phase 3: Network Reconnaissance and Perimeter Auditing (Nmap)

* 📊 Status: COMPLETED (Validated Security Posture)
* 🚀 Core Tooling: Nmap Engine (Network Mapper)

#### 🎯 Project Objectives Achieved

* 🔍 Active Host Discovery: Mapped all active devices on the local subnet using ARP and ICMP techniques.
* 🧪 Firewall Validation: Verified that UFW correctly filters all ports except SSH (22) and HTTP (80).
* 🕵️ Scan Method Comparison: Executed multiple Nmap scan types to understand their behavior and confirm the VM’s visibility.
* ✅ Security Posture Confirmation: Proved that the perimeter is locked down – only whitelisted services are exposed.

#### 🧠 Nmap Scan Types Utilised

| Scan Type | Flag | Description |
|-----------|------|-------------|
| Host Discovery (no port scan) | -sn | Ping sweep to identify live hosts |
| ARP Discovery | -PR | Layer 2 ARP ping (most reliable on LAN) |
| SYN Stealth Scan | -sS | Half‑open TCP scan (default for privileged users) |
| TCP Connect Scan | -sT | Full three‑way handshake scan |
| Skip Host Discovery | -Pn | Treat host as up, skip ping probes |
| Service/Version Detection | -sV | Probe open ports to identify service versions |
| Aggressive Scan | -A | Enables OS detection, version detection, script scanning, and traceroute |

#### 🔧 Commands Executed

# Host discovery
nmap -sn 192.168.1.0/24
nmap -PR -sn 192.168.1.0/24

# Port scans (bypassing host discovery)
nmap -Pn -sS 192.168.1.200
nmap -Pn -sT 192.168.1.200

# Service version detection
nmap -sV 192.168.1.200

#### 📊 Key Results

* Host Discovery: ARP scans (-PR) showed full visibility of the VM (MAC: 08:00:27:xx:xx:xx – VirtualBox OUI). The simple ping sweep (-sn) also detected the VM, confirming proper network binding.
* Port Exposure: Only ports 22/tcp (SSH) and 80/tcp (HTTP) were open; all other ports were reported as filtered – a direct result of the UFW default‑deny policy.
* Service Fingerprinting: Confirmed that Nginx is serving HTTP and OpenSSH is listening on port 22.

#### 🧪 Security Validation Outcome

✔ The firewall is correctly enforcing perimeter rules – only explicitly allowed services are exposed.
✔ The VM is discoverable under standard ARP‑based scans, but all non‑essential ports remain hidden.
✔ No unauthorised external exposure was detected.
✔ The Nmap results align perfectly with the intended security posture, giving confidence that the defensive controls are effective.

---

### 🔬 Phase 4: Network Packet Forensics and Traffic Analysis (Wireshark + tcpdump)

* 📊 Status: COMPLETED & VALIDATED
* 🚀 Core Tooling: Wireshark Protocol Analyzer (Windows host) + tcpdump (Ubuntu VM)

#### 🎯 Objectives Achieved

* 🤝 ICMP Packet Dissection: Successfully captured and analyzed live ICMP Echo Request/Reply traffic between the Windows host (192.168.1.37) and the Ubuntu VM (192.168.1.200).
* 🧠 Cross‑Tool Validation: Used tcpdump inside the VM to provide ground‑truth confirmation that packets were reaching the OS, eliminating any doubt about Wireshark’s accuracy.
* 🔍 Root Cause Resolution: Identified that the initial "invisibility" of VM traffic was due to incorrect interface selection in Wireshark (capturing on the wrong Ethernet adapter), not a network failure.
* 📈 Traffic Correlation: Matched Windows ping command output with real packet‑level flow in Wireshark, proving end‑to‑end Layer 3 connectivity.
* 📡 LAN Visibility: Confirmed full bidirectional communication under Bridged VirtualBox networking by comparing traffic to a physical device (phone ping), isolating the VM’s virtual network path.

#### 🧪 Key Validation Experiment

**1. ICMP Echo Test (Windows → VM)**
- Command: ping -4 192.168.1.200
- Result: ✅ 0% packet loss, stable replies, low latency (RTT <1ms to 1ms).

**2. Wireshark Capture Results**
- Observed Frames:
  - 192.168.1.37 → 192.168.1.200 ICMP Echo Request
  - 192.168.1.200 → 192.168.1.37 ICMP Echo Reply
- Conclusion: ✅ Full bidirectional ICMP flow confirmed.

**3. tcpdump Ground Truth Validation (VM)**
- Command: sudo tcpdump -n icmp
- Observed Output:
  - IP 192.168.1.37 > 192.168.1.200: ICMP echo request
  - IP 192.168.1.200 > 192.168.1.37: ICMP echo reply
- Conclusion: ✅ Confirms the VM’s network stack is receiving and replying to packets correctly.

#### 🧠 Technical Insights Learned

- Interface Dependency: Wireshark visibility depends strictly on capturing the correct physical NIC (Ethernet adapter) where traffic flows.
- Wireshark vs. Reality: "No response found" in Wireshark is often a display artifact (packet order or rendering delay) and does not indicate packet loss.
- VM Networking: In Bridged mode, the VM acts as a real LAN device, but traffic can still be isolated from the host’s view if the host captures the wrong virtual interface.
- Diagnostic Workflow: Always use tcpdump on the endpoint to confirm if the OS is actually seeing the traffic before blaming the network or Wireshark.

#### 🚨 Root Cause Resolution (Earlier Issue)

- ❌ Initial Misconception: Believed the VM’s ICMP traffic was invisible or blocked by the firewall.
- 🔍 Actual Cause: The Windows Wireshark instance was capturing on a VirtualBox adapter or a non‑active Ethernet interface.
- ✔ Resolution: Selected the correct Ethernet adapter associated with 192.168.1.37, restarted the capture, and immediately observed ICMP traffic. This was validated by tcpdump inside the VM.

#### 🧪 Security Relevance

This phase establishes the foundational visibility required for:
- Intrusion detection (seeing malicious packets in real‑time).
- Network troubleshooting (isolating Layer 3 issues).
- Correlating packet‑level events with SIEM logs (Phase 5).

---

### 📊 Phase 5: Distributed Enterprise SIEM Data Analytics (Splunk)

* 📊 Status: SLATED FOR EXECUTION
* 🚀 Core Tooling: Splunk Enterprise Core / Universal Forwarder Architecture

#### 🎯 Project Objectives

* 🗄️ Centralised Aggregation: Spin up a secondary Virtual Machine running Splunk Enterprise to act as a centralised Security Operations Center (SOC) hub.
* 🚚 Forwarder Provisioning: Deploy lightweight Splunk Universal Forwarders to pipeline /var/log/nginx/access.log and /var/log/fail2ban.log securely to the indexer.
* 📈 Dashboard Engineering: Write custom Splunk Processing Language (SPL) searches to build live dashboard visualisations mapping geo‑IP intelligence and firewall drop counts.
* 🚨 Alerting Pipeline: Configure real‑time alerts for high‑frequency scanning events or repeated ban triggers.

---

### 🛠️ Engineering Problems Encountered, Troubleshooting and Incident Logs

#### 1. 🛑 Problem: VirtualBox Installation Blocking (Missing Dependencies)
* 💻 Symptom: VirtualBox installer warned about missing core packages regarding Python and pywin32 components.
* ✅ Resolution: Installed Python runtime environment and executed "pip install pywin32" on the Windows host machine before running installer.

#### 2. 🛑 Problem: Hypervisor Partition Failure (VERR_SVM_DISABLED)
* 💻 Symptom: VirtualBox threw an execution block code E_FAIL (0x80004005).
* ✅ Resolution: Rebooted JayPC, entered the BIOS menu via the Delete key, navigated to Advanced CPU Configuration, and toggled SVM Mode to Enabled.

#### 3. 🛑 Problem: Ubuntu Server Installation Long Loading / Hang
* 💻 Symptom: The operating system installation screen remained stuck during package fetching.
* ✅ Resolution: Increased VM hardware allocation to 2 vCPU Cores and 2GB RAM to handle compilation overhead over the Bridged network adapter.

#### 4. 🛑 Problem: Managing Open Access Risk vs Perimeter Lockdown
* 💻 Symptom: Initial firewall rule allowing port 22 exposed the SSH login interface globally to any local node connected across the network subnet.
* ✅ Resolution: Cleared global inbound rules (sudo ufw delete 1) and provisioned discrete IP‑matching access vectors using the dedicated interface addresses of the management computers.

#### 5. 🛑 Problem: Administrative Lockout via Client DHCP Lease Rotation
* 💻 Symptom: Client management machines (MacBook and Windows PC) were abruptly blocked from SSH access after the local home router dynamically rotated their IP assignments, invalidating the firewall's strict single‑IP whitelist.
* ✅ Resolution: Accessed the core hypervisor console directly and expanded the network perimeter rule from a single host IP structure to a local subnet mask configuration: "sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp". This maintains strict external blocking while allowing dynamic internal mobility.

#### 6. 🛑 Problem: Router Admin Dashboard Lockout (Unknown Credentials)
* 💻 Symptom: Attempted to configure permanent DHCP IP Reservations via the default gateway (192.168.1.1), but was completely blocked due to unconfigured router administrative login credentials.
* ✅ Resolution: Pivoted away from perimeter infrastructure modifications and shifted focus toward a host‑based remediation strategy managed directly within the OS.

#### 7. 🛑 Problem: Server IP Rotation on System Restart
* 💻 Symptom: Every time the Ubuntu server instance rebooted, the home router assigned it a different dynamic IP address (e.g., shuffling from 192.168.1.39), requiring manual console checks (ip a) before clients could connect.
* ✅ Resolution: Decoupled the server from the router's dynamic pool by rewriting the network configuration file (/etc/netplan/50-cloud-init.yaml) to inject a persistent, static IP binding at 192.168.1.200/24.

#### 8. 🛑 Problem: Netplan Configuration Exposure Vulnerability
* 💻 Symptom: Executing network configuration updates via "sudo netplan try" threw security warnings stating permissions for the file were too open.
* ✅ Resolution: Hardcoded strict file‑system security by modifying file permissions to read/write exclusively for root system admin: "sudo chmod 600 /etc/netplan/50-cloud-init.yaml".

#### 9. 🛑 Problem: SSH Host Identification Fingerprint Mismatch
* 💻 Symptom: After applying the static IP transition, client machines blocked connection attempts with a critical security alert: "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!".
* ✅ Resolution: Cleared outdated cryptographic keys mapped to the target IP address in local machine records by executing "ssh-keygen -R 192.168.1.200" (and 192.168.1.39) on the client terminals, forcing a clean key re‑exchange.

#### 10. 🛑 Problem: Active Bypass of Firewall Ban Entries
* 💻 Symptom: Even after the automated system successfully verified an attack signature and injected an active rule into UFW, the browser on the client machine could still reload and retrieve valid 404 pages.
* ✅ Resolution: Isolated the root cause to TCP Keep‑Alive Session Persistence. The browser pushed traffic through a persistent communication socket established before the firewall rule took effect. Resolved by manually triggering the socket statistics utility to sever the connection: "sudo ss -K dst 192.168.1.2".

#### 11. 🛑 Problem: Persistence of Application Error Screens Post‑Ban
* 💻 Symptom: When modifying the system to deploy an absolute packet‑suppression rule (action = ufw[action=deny]), the client test machine continued to display the immediate Nginx 404 Not Found screen instead of a timeout connection error.
* ✅ Resolution: Isolated root cause to Client‑Side Browser Caching. The browser pulled the static 404 error page elements straight out of local storage. Resolved by running validation checks inside an Incognito Environment to force raw wire frames, verifying the true "ERR_CONNECTION_TIMED_OUT" state.

#### 12. 🛑 Problem: Authentication Failures via HTTPS Git Token Operations
* 💻 Symptom: Attempts to run "git push" over the standard HTTPS remote protocol yielded invalid credential errors due to personal access token restrictions.
* ✅ Resolution: Completely decoupled the repository from HTTPS operations by generating an asymmetric Ed25519 SSH Key Pair ("ssh-keygen -t ed25519") on the host system, binding the public key directly to GitHub, and switching the remote configuration link over to the secure SSH routing architecture: "git remote add origin git@github.com:Jaysese125/Active-Defense-Ubuntu-Lab.git".

#### 13. 🛑 Problem: Commits Failing to Register on GitHub Contribution Dashboard
* 💻 Symptom: Code modifications were successfully accepted by the remote repository, but failed to log contribution metrics on the user account history timeline.
* ✅ Resolution: Traced behavior to a localized typo in the shell parameter where the local identity address ("git config user.email") mismatched the verified profile. Reconfigured utilizing "git config --global user.email "your@email.com"" to precisely align tracking metrics.

#### 14. 🛑 Problem: Git Push Rejection via Out‑of‑Sync Remote Timeline
* 💻 Symptom: Executing "git push" threw a critical error: "! [rejected] main -> main (fetch first)", indicating the remote branch had commits missing from local storage.
* ✅ Resolution: Synchronized the divergent branches by pulling down remote states and forcing a local non‑rebase merge event using "git pull origin main --no-rebase" before executing a clean upstream deployment.

#### 15. 🛑 Problem: Nmap Host Discovery Not Showing the VM
* 💻 Symptom: Initial "nmap -sn" scans sometimes did not list the Ubuntu VM, causing confusion about network visibility.
* ✅ Resolution: Understood that -sn relies on ICMP echo and TCP SYN to port 443; the VM's firewall may block these. Switching to ARP‑based discovery (-PR -sn) reliably detected the VM because ARP operates at Layer 2 and is not filtered by UFW.

#### 16. 🛑 Problem: Nmap Interface Selection Failure on Windows
* 💻 Symptom: Specifying a network interface with "-e Wi-Fi" or "-e Ethernet" produced errors due to incorrect interface naming in the Windows Npcap driver.
* ✅ Resolution: Used "nmap --iflist" to list all available interfaces and selected the correct one (e.g., "-e "Wi-Fi"" or "-e "Ethernet"" as shown in the list). Alternatively, omitted the -e flag to let Nmap auto‑select the active interface.

#### 17. 🛑 Problem: Misunderstanding of -Pn Behavior
* 💻 Symptom: Expected -Pn to force a port scan regardless of host status, but it was used incorrectly alongside -sn (which disables port scanning entirely).
* ✅ Resolution: Learned that -sn is a pure host‑discovery flag that skips port scanning; -Pn disables host discovery but still performs port scanning. Used "-Pn -sS" together to scan a target presumed alive without preliminary ping probes.

#### 18. 🛑 Problem: Wireshark Interface Selection Confusion (Phase 4)
* 💻 Symptom: ICMP packets were visible in tcpdump on the VM and in ping replies, but Wireshark on Windows showed nothing for the icmp filter.
* ✅ Resolution: Identified that Wireshark was capturing on a VirtualBox adapter or a non‑active Ethernet interface. Switched to the correct Ethernet adapter associated with the host IP (192.168.1.37), restarted capture, and immediately observed bidirectional ICMP traffic. Cross‑validated with tcpdump to confirm the OS was indeed receiving the packets.
