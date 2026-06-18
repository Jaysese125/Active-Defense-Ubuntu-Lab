===============================================================================
               ENTERPRISE CYBERSECURITY ENGINEERING HOME LAB
                           ENGINEERING JOURNAL
===============================================================================

-------------------------------------------------------------------------------
                       THE MASTER ARCHITECTURE BLUEPRINT
-------------------------------------------------------------------------------
Phase 1: Hardening    ➔ Phase 2: Active IPS    ➔ Phase 3: Recon/Audit ➔ Phase 4: Forensics ➔ Phase 5: Splunk SIEM
(COMPLETED)             (COMPLETED)              (NEXT STEP)            (FUTURE STEP)          (FUTURE STEP)


-------------------------------------------------------------------------------
         PHASE 1: LOCAL LINUX SERVER DEPLOYMENT & BASELINE HARDENING
-------------------------------------------------------------------------------
🛰️ Status:                  100% COMPLETE & VERIFIED
🖥️ Host Machine:            JayPC (AMD Ryzen 3 3200G, 16GB RAM)
⚙️ Hypervisor:              Oracle VirtualBox
🐧 Guest OS:                Ubuntu Server 24.04 LTS (Headless Setup)
📌 Dedicated Management IP: 192.168.1.200

Project Milestones Achieved:
* 🧬 Hardware Virtualization Configuration: Enabled SVM Mode in the AMD BIOS to 
  unlock hypervisor capabilities on the host system.
* 🎛️ Resource Allocation: Provisioned a balanced, headless guest instance 
  utilizing 2 vCPU Cores and 2GB RAM.
* 🌐 Layer 2 Networking Architecture: Configured the VirtualBox network interface 
  card to Bridged Mode to seamlessly attach the server to the physical home subnet.
* 🔑 Secure Access Implementation: Installed and hardened OpenSSH Server components 
  to handle administrative connectivity.
* 🛑 Zero-Trust Administrative Management: Established encrypted, concurrent SSH 
  terminal sessions from specialized client machines (Windows PC and MacBook Air).
* 🧱 Perimeter Hardening: Deployed Uncomplicated Firewall (UFW) enforcing a 
  default-deny inbound posture.
* 🎯 Network Whitelisting: Migrated the perimeter access rule from a globally 
  exposed network configuration to a targeted local subnet mask whitelist 
  (192.168.1.0/24), neutralizing unauthorized access vectors.
* ⚡ Static IP Orchestration: Overrode the dynamic host configuration protocol 
  (DHCP) by injecting a permanent static routing table profile via Netplan 
  configuration.


-------------------------------------------------------------------------------
         PHASE 2: MANUAL LOG ANALYSIS & ACTIVE INTRUSION PREVENTION (IPS)
-------------------------------------------------------------------------------
🛰️ Status:               100% COMPLETE & VERIFIED
🚀 Core Software Engine: Nginx Web Server Ecosystem
🛡️ IPS Daemon:           Fail2Ban System Automation

Project Milestones Achieved:
* 🔍 Forensic Log Parsing Infrastructure: Developed multi-column parsing pipelines 
  using Linux data manipulation tools to inspect real-time application access telemetry.
* 🧠 Pattern Discovery: Isolated automated directory discovery brute-force signatures 
  by mapping network communication behaviors to specific internal server resources.
* ⚡ Automated Threat Suppression: Implemented a custom regular expression filter 
  linked to a dynamic Fail2Ban jail policy to automatically manipulate the host 
  firewall state upon threat detection.
* 🔨 Perimeter Policy Hardening: Configured strict, unnegotiated packet dropping 
  mechanisms to fully neutralize active network scanners.

DEEP-DIVE: PHASE 2 INCIDENT ANALYSIS & TELEMETRY

1. Forensic Log Parsing (The Command-Line Pipeline)
To isolate unauthorized scanning behavior, specific text manipulation pipelines 
were executed directly against the live Nginx access log (/var/log/nginx/access.log):

* Source Isolation (Tracking Hit Volume per Host):
  sudo awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr

* URI Intent Analysis (Tracking Targeted Directory Paths):
  sudo awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -nr

* The Cross-Reference Matrix (Tying Unique Hosts to Action Profiles):
  sudo awk '{print $1, $7}' /var/log/nginx/access.log | sort | uniq -c | sort -nr

📊 Analytical Findings:
The output explicitly mapped out two distinct operational profiles across the 
local network fabric:
1. 192.168.1.41 (Mobile Endpoint): Valid user behavior interacting exclusively 
   with the site root directory (/).
2. 192.168.1.2 (Windows Endpoint): Aggressive, high-velocity directory traversal 
   scanner hunting for administrative entry points (/wp-login.php, 
   /secret-passwords.txt, /admin), generating a continuous series of HTTP 404 
   Not Found errors.

2. Automated Intrusion Prevention System (IPS) Configuration
To programmatically block the malicious profile, an automated active response 
loop was deployed:

* The Signature Filter (/etc/fail2ban/filter.d/nginx-404.conf):
  [Definition]
  failregex = ^<HOST> - - \\[.*\\] ".*" 404 .*

* The Enforcement Jail Configuration (/etc/fail2ban/jail.d/nginx-404.local):
  Configured a strict operational policy: If an IP triggers 5 failed resource 
  requests (404) inside a rolling 10-second window, Fail2Ban dynamically modifies 
  the kernel firewall layer to inject a hard network block for a 10-minute duration:
  
  [nginx-404]
  enabled = true
  port    = http,https
  filter  = nginx-404
  logpath = /var/log/nginx/access.log
  backend = auto
  maxretry = 5
  findtime = 10
  bantime  = 600
  action   = ufw[action=deny]


-------------------------------------------------------------------------------
            PHASE 3: NETWORK RECONNAISSANCE & PERIMETER AUDITING (NMAP)
-------------------------------------------------------------------------------
🛰️ Status:       SLATED FOR EXECUTION (Next Step)
🚀 Core Tooling: Nmap Engine (Network Mapper)

Project Objectives:
* 🎯 Threat Modeling: Conduct aggressive stealth TCP Syn scans (nmap -sS -p- -A) 
  from client environments to map open ports and evaluate edge exposure.
* 🧪 Validation Vector: Verify firewall efficacy by proving that unwhitelisted 
  targets see the server as completely filtered (invisible) while whitelist 
  clients can communicate transparently.


-------------------------------------------------------------------------------
            PHASE 4: NETWORK PACKET FORENSICS & TRAFFIC ANALYSIS (WIRESHARK)
-------------------------------------------------------------------------------
🛰️ Status:       SLATED FOR EXECUTION
🚀 Core Tooling: Wireshark Protocol Analyzer

Project Objectives:
* 🤝 Handshake Dissection: Capture live packet frames over the Bridged interface to 
  inspect raw 3-Way TCP handshakes (SYN, SYN-ACK, ACK).
* 🔬 Threat Signature Mapping: Inspect network packet flags and raw hex bytes 
  during simulated scanning loops to analyze attack patterns before they hit logs.


-------------------------------------------------------------------------------
            PHASE 5: DISTRIBUTED ENTERPRISE SIEM DATA ANALYTICS (SPLUNK)
-------------------------------------------------------------------------------
🛰️ Status:       SLATED FOR EXECUTION
🚀 Core Tooling: Splunk Enterprise Core / Universal Forwarder Architecture

Project Objectives:
* 🗄️ Centralized Aggregation: Spin up a secondary Virtual Machine running Splunk 
  Enterprise to act as a centralized Security Operations Center (SOC) hub.
* 🚚 Forwarder Provisioning: Deploy lightweight Splunk Universal Forwarders to 
  pipeline /var/log/nginx/access.log and /var/log/fail2ban.log securely to the indexer.
* 📈 Dashboard Engineering: Write custom Splunk Processing Language (SPL) searches 
  to build live dashboard visualizations mapping geo-IP intelligence and firewall 
  drop counts.


===============================================================================
       ENGINEERING PROBLEMS ENCOUNTERED, TROUBLESHOOTING & INCIDENT LOGS
===============================================================================

1. 🛠️ Problem Encountered: VirtualBox Installation Blocking (Missing Dependencies)
   * Symptom: VirtualBox installer warned about missing core packages regarding 
     Python and pywin32 components.
   * Resolution: Installed Python runtime environment and executed 
     "pip install pywin32" on the Windows host machine before running installer.

2. 🛠️ Problem Encountered: Hypervisor Partition Failure (VERR_SVM_DISABLED)
   * Symptom: VirtualBox threw an execution block code E_FAIL (0x80004005).
   * Resolution: Rebooted JayPC, entered the BIOS menu via the Delete key, 
     navigated to Advanced CPU Configuration, and toggled SVM Mode to Enabled.

3. 🛠️ Problem Encountered: Ubuntu Server Installation Long Loading / Hang
   * Symptom: The operating system installation screen remained stuck during 
     package fetching.
   * Resolution: Increased VM hardware allocation to 2 vCPU Cores and 2GB RAM 
     to handle compilation overhead over the Bridged network adapter.

4. 🛠️ Problem Encountered: Managing Open Access Risk vs Perimeter Lockdown
   * Symptom: Initial firewall rule allowing port 22 exposed the SSH login 
     interface globally to any local node connected across the network subnet.
   * Resolution: Cleared global inbound rules (sudo ufw delete 1) and 
     provisioned discrete IP-matching access vectors using the dedicated 
     interface addresses of the management computers.

5. 🛠️ Problem Encountered: Administrative Lockout via Client DHCP Lease Rotation
   * Symptom: Client management machines (MacBook and Windows PC) were abruptly 
     blocked from SSH access after the local home router dynamically rotated 
     their IP assignments, invalidating the firewall's strict single-IP whitelist.
   * Resolution: Accessed the core hypervisor console directly and expanded the 
     network perimeter rule from a single host IP structure to a local subnet 
     mask configuration: "sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp"
     This maintains strict external blocking while allowing dynamic internal mobility.

6. 🛠️ Problem Encountered: Router Admin Dashboard Lockout (Unknown Credentials)
   * Symptom: Attempted to configure permanent DHCP IP Reservations via the default 
     gateway (192.168.1.1), but was completely blocked due to unconfigured router 
     administrative login credentials.
   * Resolution: Pivoted away from perimeter infrastructure modifications and shifted 
     focus toward a host-based remediation strategy managed directly within the OS.

7. 🛠️ Problem Encountered: Server IP Rotation on System Restart
   * Symptom: Every time the Ubuntu server instance rebooted, the home router 
     assigned it a different dynamic IP address (e.g., shuffling from 192.168.1.39), 
     requiring manual console checks (ip a) before clients could connect.
   * Resolution: Decoupled the server from the router's dynamic pool by rewriting 
     the network configuration file (/etc/netplan/50-cloud-init.yaml) to inject 
     a persistent, static IP binding at 192.168.1.200/24.

8. 🛠️ Problem Encountered: Netplan Configuration Exposure Vulnerability
   * Symptom: Executing network configuration updates via "sudo netplan try" threw 
     security warnings stating permissions for the file were too open.
   * Resolution: Hardcoded strict file-system security by modifying file permissions 
     to read/write exclusively for root system admin: 
     "sudo chmod 600 /etc/netplan/50-cloud-init.yaml".

9. 🛠️ Problem Encountered: SSH Host Identification Fingerprint Mismatch
   * Symptom: After applying the static IP transition, client machines blocked 
     connection attempts with a critical security alert: "WARNING: REMOTE HOST 
     IDENTIFICATION HAS CHANGED!".
   * Resolution: Cleared outdated cryptographic keys mapped to the target IP address 
     in local machine records by executing "ssh-keygen -R 192.168.1.200" (and 
     192.168.1.39) on the client terminals, forcing a clean key re-exchange.

10. 🛠️ Problem Encountered: Active Bypass of Active Firewall Ban Entries
    * Symptom: Even after the automated system successfully verified an attack 
      signature and injected an active rule into UFW, the browser on the client 
      machine could still reload and retrieve valid 404 pages.
    * Resolution: Isolated the root cause to TCP Keep-Alive Session Persistence. 
      The browser pushed traffic through a persistent communication socket established 
      before the firewall rule took effect. Resolved by manually triggering the 
      socket statistics utility to sever the connection: "sudo ss -K dst 192.168.1.2".

11. 🛠️ Problem Encountered: Persistence of Application Error Screens Post-Ban
    * Symptom: When modifying the system to deploy an absolute package suppression 
      rule (action = ufw[action=deny]), the client test machine continued to display 
      the immediate Nginx 404 Not Found screen instead of a timeout connection error.
    * Resolution: Isolated root cause to Client-Side Browser Caching. The browser 
      pulled the static 404 error page elements straight out of local storage. 
      Resolved by running validation checks inside an Incognito Environment to 
      force raw wire frames, verifying the true "ERR_CONNECTION_TIMED_OUT" state.

12. 🛠️ Problem Encountered: Authentication Failures via HTTPS Git Token Operations
    * Symptom: Attempts to run "git push" over the standard HTTPS remote protocol 
      yielded invalid credential errors due to personal access token restrictions.
    * Resolution: Completely decoupled the repository from HTTPS operations by 
      generating an asymmetric Ed25519 SSH Key Pair ("ssh-keygen -t ed25519") on 
      the host system, binding the public key directly to GitHub, and switching 
      the remote configuration link over to the secure SSH routing architecture: 
      "git remote add origin git@github.com:Jaysese125/Active-Defense-Ubuntu-Lab.git".

13. 🛠️ Problem Encountered: Commits Failing to Register on GitHub Contribution Dashboard
    * Symptom: Code modifications were successfully accepted by the remote repository, 
      but failed to log contribution metrics on the user account history timeline.
    * Resolution: Traced behavior to a localized typo in the shell parameter where 
      the local identity address ("git config user.email") mismatched the verified 
      profile. Reconfigured utilizing 'git config --global user.email "your@email.com"' 
      to precisely align tracking metrics.

14. 🛠️ Problem Encountered: Git Push Rejection via Out-of-Sync Remote Timeline
    * Symptom: Executing "git push" threw a critical error: "! [rejected] main -> main 
      (fetch first)", indicating the remote branch had commits missing from local storage.
    * Resolution: Synchronized the divergent branches by pulling down remote states 
      and forcing a local non-rebase merge event using "git pull origin main --no-rebase" 
      before executing a clean upstream deployment.
===============================================================================
