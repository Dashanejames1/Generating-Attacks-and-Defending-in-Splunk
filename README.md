# Generating-Attacks-and-Defending-in-Splunk


**Author:** Dashane James  
**Lab Environment:** [e.g. VMware Workstation | Kali Linux | Metasploitable 2]  
**Purpose:** [The goal of this repository is to demonstrate the core SOC analyst workflow of generating attack traffic, ingesting logs into a SIEM, and building detection rules to identify threats. Rather than simply running tools, the focus is on the defensive side. Understanding what attack patterns look like in log data and translating that into actionable Splunk alerts.]
**Status:** 🔵 Completed

---

## 📋 Overview

[This repository documents a hands-on SOC analyst simulation where real attack traffic was generated in a controlled lab environment and detected using Splunk SIEM. I performed two distinct attack types a network port scan and an SSH brute force attack while simultaneously capturing all network traffic with TCPDump. The captured data was then imported into Splunk where I built SPL detection queries and configured automated alerts to identify both attack patterns. This project demonstrates the complete SOC analyst workflow: generate attack traffic, ingest logs into a SIEM, write detection logic, and set alert thresholds ]

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation (Host-Only Network) |
| Attacker Machine | Kali Linux 2026.1 — `192.168.79.129` |
| Target Machine | [Metasploitable 2] — `192.168.79.130` |
| Network Type | Host-Only (isolated, no internet exposure) |
| Host OS | Windows 11 — ASUS Vivobook 14 |

> ⚠️ **Note:** All activity was performed in a controlled, isolated lab environment against deliberately vulnerable machines. No unauthorized access to live networks was performed.

---

## 🛠️ Tools Used

- **Nmap** — Used for both standard scanning and running NSE (Nmap Scripting Engine) scripts to detect and, in one case, actively exploit vulnerabilities
- **NSE (Nmap Scripting Engine)** — Ran default (`-sC`), targeted (`ftp-anon`, `smb-vuln*`), and broad (`vuln`) script categories against the target
- **NVD (National Vulnerability Database)** — Referenced to research CVE details, CVSS scores, and attack vectors for confirmed vulnerabilities

---

## 📁 Repository Structure

```
[Repo-Name]/
├── README.md
├── [folder-1]/
│   ├── [file1.txt]        # Description of file
│   └── [file2.md]         # Description of file
├── [folder-2]/
│   ├── [file3.md]         # Description of file
│   └── [file4.txt]        # Description of file
└── reports/
    └── [final-report.md]  # Full assessment report
```

---

## 🔬 Tasks / Assessments Performed

### 1. [Run Nmap scan from Kali while capturing with TCPDump]

[This tasks required that I generate real network reconnaissance traffic while simultaneously capturing all packets at the network level. This simulates what an attacker's port scan looks like from a packet capture perspective and produces the raw data needed for SIEM analysis.]


# Command used
[sudo tcpdump -i eth0 -w ~/lab_capture.pcap] -tcp dump

<img width="320" height="184" alt="image" src="https://github.com/user-attachments/assets/4e6c8a16-8562-422e-9b5f-3c80357e89e5" />


[sudo nmap -sT -Pn --disable-arp-ping -n 192.168.79.130] - nmap

<img width="346" height="312" alt="image" src="https://github.com/user-attachments/assets/b5a7ba69-4468-438e-a7fd-9fbc2f266900" />

# Output

For this task I opened two terminal windows simultaneously. In the first terminal I started a TCPDump capture using `sudo tcpdump -i eth0 -w ~/lab_capture.pcap`, which began listening on the eth0 network interface and saving all captured packets to a file called lab_capture.pcap. In the second terminal I ran an Nmap scan against Metasploitable (192.168.79.130) while TCPDump was actively capturing in the background. The Nmap scan returned a full list of open ports and services on the target which confirmed the target was live and its attack surface was fully exposed. Once the Nmap scan completed I stopped the TCPDump capture with Ctrl+C, which saved all the network traffic generated during the scan to lab_capture.pcap. This file contains the raw packet-level evidence of the port scan and will be imported into Splunk for analysis. This simulates how a SOC analyst would capture and investigate suspicious network reconnaissance activity.




### 2. [Run SSH brute force with Hydra/Medusa against Metaspolitable port 22.]

[Objective: Simulate a credential-based attack against the target's SSH service to generate authentication attempt logs. This produces the brute force traffic pattern that SOC analysts are trained to detect and investigate.]

# Commands used
[medusa -h 192.168.79.130 -u msfadmin -P /usr/share/wordlists/rockyou.txt -M ssh -t 4]

<img width="323" height="257" alt="image" src="https://github.com/user-attachments/assets/6411f28a-da43-4d02-8332-f338b1eab575" />


[echo -e "msfadmin\nadmin\npassword\n123456\nroot\ntoor" > ~/quick_wordlist.txt]

<img width="323" height="38" alt="image" src="https://github.com/user-attachments/assets/8feb642d-82b9-4e8a-9402-b6b1ff0431a8" />


[medusa -h 192.168.79.130 -u msfadmin -P ~/quick_wordlist.txt -M ssh -t 4]

<img width="322" height="148" alt="image" src="https://github.com/user-attachments/assets/8c503311-3d47-459d-b69d-b820af803216" />


# Output Explained

During this task I initially attempted to use Hydra, which is the industry standard tool for this type of attack. However Hydra failed with a kex error — a cryptographic key exchange failure caused by an incompatibility between Hydra's modern SSH library (libssh) and Metasploitable's outdated SSH server.Since there was zero overlap between the algorithms both sides support, the connection failed before a single password attempt could be made.

Next I decided to move on to attempting this task with Medusa rather than Hydra because it uses a different underlying SSH implementation that retains support for legacy algorithms, allowing it to negotiate a connection with Metasploitable's old SSH server. However the initial Medusa run using the full rockyou.txt wordlist (14 million passwords) proved impractical. After running for over 20 minutes and reaching 3,000 attempts without a result, I switched to a targeted wordlist containing the most commonly used default credentials.

Using Medusa with a targeted wordlist, the SSH brute force attack against Metasploitable (192.168.79.130) successfully cracked the credentials on the first attempt — username: msfadmin, password: msfadmin. This confirms the target is running default credentials with no account lockout policy, meaning an attacker can attempt unlimited logins without being blocked. In a real environment this would be flagged as two critical findings: default credentials in use and no brute force protection on SSH.



### 3. [Import TCPDump capture into Splunk]
[Ingest the raw network capture data into the SIEM to make it searchable and queryable. This step converts raw packet data into indexed events that SPL queries can run against — the foundation of all SIEM-based detection.] 


# Command used
[tcpdump -r ~/lab_capture.pcap -nn -tttt > ~/lab_capture.txt] - converting pcap file to readable format

<img width="423" height="33" alt="image" src="https://github.com/user-attachments/assets/9c7a0e47-49c1-433d-ab55-28ddb98d8b44" />


[index=main source="Lab_capture.txt"] -Splunk Search Filter

<img width="856" height="340" alt="image" src="https://github.com/user-attachments/assets/82fb9b77-68b0-497f-9549-9fb540dc5b43" />


# Output

After running the TCPDump capture as seen in the first screenshot, I converted the binary pcap file to a human-readable text format using tcpdump -r ~/lab_capture.pcap -nn -tttt > ~/lab_capture.txt. The converted file was 368KB containing all network packets captured during the Nmap scan. The second screenshot shows Splunk parsing real network events from lab_capture.txt including IP addresses, ARP requests and replies, and precise timestamps. This confirms Splunk is now ingesting real attack traffic data generated from the lab environment



### 4. [Write the SPL to detect the port scan pattern]

[Objective: Build a detection query that identifies port scan activity from the ingested network data. Using regex extraction and event counting, the query surfaces IP addresses generating abnormally high connection volumes the defining characteristic of a port scan.]


#Command/Splunk Search

[index=main source="lab_capture.txt" | stats count by host | sort -count] - splunk search

<img width="856" height="344" alt="image" src="https://github.com/user-attachments/assets/42515952-a0d3-4005-ab27-971c8ebe7ec7" />

[index=main source="lab_capture.txt" | rex field=_raw "(?<src_ip>\d+\.\d+\.\d+\.\d+)" | stats count by src_ip | sort -count]

<img width="857" height="238" alt="image" src="https://github.com/user-attachments/assets/ef7a7048-3cfa-4f78-9a1a-8010c1cb545e" />

# Output

To detect the port scan pattern I ran two SPL searches against the imported TCPDump capture file. The first search index=main source="lab_capture.txt" | stats count by host | sort -count confirmed the data was successfully ingested into Splunk and identified the host machine the capture originated from.

The second search used a regex pattern to extract all IP addresses from the raw TCPDump text and count events per IP: index=main source="lab_capture.txt" | rex field=_raw "(?<src_ip>\d+\.\d+\.\d+\.\d+)" | stats count by src_ip | sort -count. This returned 2,571 total events across 4 IP addresses. Kali (192.168.79.129) generated 1,273 events and Metasploitable (192.168.79.130) generated 1,006 events — accounting for the vast majority of all captured traffic.

**Findings**

Together these two searches demonstrate the core SOC analyst workflow — first confirm your data is ingested correctly, then build detection queries on top of it to surface attack patterns. In a real environment this second query would be built into a scheduled alert that fires whenever a single source IP exceeds a defined event threshold within a short time window, giving analysts an automated early warning system for reconnaissance activity.




### 5. [Write SPL to detect brute force attempts]

[Objective: Build a detection query that identifies brute force activity from the Medusa output logs. The query extracts credential attempt data and surfaces successful account compromises, demonstrating how a SOC analyst would use Splunk to investigate a suspected brute force incident.]

#commands used:

[medusa -h 192.168.79.130 -u msfadmin -P ~/quick_wordlist.txt -M ssh -t 4 -O ~/medusa_results.txt]

<img width="672" height="87" alt="image" src="https://github.com/user-attachments/assets/e2e44af0-b987-4177-a005-3543a4caed54" />

<img width="803" height="296" alt="image" src="https://github.com/user-attachments/assets/c551f42e-13f2-4df8-8b75-b0ea023b722b" />
basic detection

<img width="853" height="212" alt="image" src="https://github.com/user-attachments/assets/22bbfd36-9b7a-4380-a723-3c27eea3fe00" />
Password extraction

### Output

First, Running index=main source="medusa_results.txt" "ACCOUNT FOUND" returned exactly 1 event — the precise moment the brute force attack succeeded. Splunk filtered through all the Medusa log data and surfaced only the critical event: a successful SSH credential crack against 192.168.79.130 at 12:54:29, confirming username msfadmin with password msfadmin. This is exactly how a SOC analyst would use Splunk during an incident investigation.

The second detection search used a regex(?) pattern to extract the cracked password field from the raw Medusa log data and display it in a structured table alongside its timestamp. Splunk successfully extracted the password 'msfadmin' from the ACCOUNT FOUND event at 12:54:29, demonstrating how SPL can parse unstructured tool output into actionable intelligence. In a real SOC environment this type of search would be used during incident response to quickly identify which credentials were compromised during a brute force attack, allowing the security team to immediately force password resets on affected accounts.



### 6. [Set Appropriate alert thresholds for each.]

[Objective: Convert the detection queries into automated scheduled alerts with appropriate trigger conditions. This closes the SOC analyst workflow loop. Moving from manual investigation to automated detection that proactively notifies analysts when attack patterns are observed.]

#commands used:

[index=main source="lab_capture.txt" | rex field=_raw "(?<src_ip>\d+\.\d+\.\d+\.\d+)" | stats count by src_ip | where count > 100]

<img width="579" height="242" alt="image" src="https://github.com/user-attachments/assets/a9012361-e144-4797-aaf2-e6f597a0b9c3" />

<img width="697" height="265" alt="Screenshot 2026-09-24 204709" src="https://github.com/user-attachments/assets/61142023-7505-4248-8894-f44d943e8a07" />


[index=main source="medusa_results.txt" | stats count as total_events, count(eval(match(_raw,"ACCOUNT CHECK"))) as attempts, count(eval(match(_raw,"SUCCESS"))) as successful_cracks]

<img width="860" height="184" alt="Screenshot 2026-09-24 212306" src="https://github.com/user-attachments/assets/1f05bacd-6836-4a83-9a22-e0d290e03850" />

<img width="638" height="152" alt="Screenshot 2026-09-24 212919" src="https://github.com/user-attachments/assets/bea2cd16-309a-4a6c-81c6-696cdbc3e6f2" />


Output:

After confirming the port scan detection search returned results showing IPs exceeding the 100 event threshold, I saved the query as a scheduled alert in Splunk titled 'Port Scan Detection'. The alert is configured to run hourly and triggers when the number of results is greater than 0 — meaning any IP generating more than 100 events within the capture data will fire the alert. The action is set to 'Add to Triggered Alerts' which logs the event in Splunk's alert dashboard for analyst review.

The threshold of 100 events was chosen deliberately — normal network traffic between two hosts rarely exceeds this volume in a short window, but a port scan generating thousands of connection attempts will immediately exceed it. In a real SOC environment this alert would be the first line of defense against reconnaissance activity, giving analysts early warning that a host is being mapped before an attacker moves to the exploitation phase.

The brute force detection search analyzed the Medusa output file showing 3 total events from the SSH brute force session. The alert was configured to run every hour and trigger when any results are returned — in a production environment with real authentication logs this same query would detect brute force attempts in near real-time, alerting the SOC team before an attacker can successfully crack credentials.

**Final Findings:** 

This lab completed the full SOC analyst workflow — generate attack traffic, ingest it into a SIEM, build detection queries, and set automated alerts.

One of the most important takeaways connects directly to the previous repository. In the SPL Search Language lab, searches like index=main | stats count by src_ip returned no results — not because the queries were wrong, but because there was nothing meaningful to search through. A static Nmap text file doesn't contain authentication events or structured network fields. This lab fixes that by generating real attack traffic first. The result: 2,571 events from a live port scan and a successful SSH credential crack, all searchable in Splunk. The lesson — a SIEM is only as powerful as the data feeding it. Perfect detection queries mean nothing without the right log sources ingested.

Two attacks were executed and detected. The Nmap port scan produced a clear signature — Kali generating 1,273 events against Metasploitable in a short window — which triggered the Port Scan Detection alert. The Medusa brute force cracked msfadmin:msfadmin in seconds, exposing two critical findings: default credentials and no account lockout policy.

Real-world problem solving was also part of this lab. Hydra failed due to a cryptographic incompatibility with Metasploitable's legacy SSH server — a situation that required switching tools over to Medusa and adapting the approach accordingly.


a SOC analyst would use Splunk during an incident investigation by searching for specific indicators like 'ACCOUNT FOUND' or 'SUCCESS' to quickly identify which accounts were compromised without manually reading through thousands of log lines."




Port / Service / Vulnerability / NSE Script / Severity / Notes

21/tcp (FTP) - ftp-vsftpd-backdoor (CVE-2011-2523)	🔴 Critical	Live exploitation confirmed. The vulnerable vsFTPd 2.3.4 backdoor allowed remote command execution and root shell access. (CVSS 9.8)

22/tcp (SSH) - Weak SSH cryptographic configuration - 🟡 Medium	Weak or outdated cryptographic algorithms were identified, potentially reducing the security of encrypted communications.

23/tcp (Telnet) - Telnet service detected	🟠 High	Telnet transmits credentials in plaintext, making usernames and passwords susceptible to interception.

80/tcp (HTTP) - Slowloris Denial-of-Service	🟡 Medium	The web server appears susceptible to the Slowloris DoS attack, which can exhaust server connections and deny service to legitimate users.

80/tcp (HTTP) - HTTP TRACE method enabled	🟢 Low	TRACE is enabled, which may aid reconnaissance or cross-site tracing attacks but does not directly compromise the server.

80/tcp (HTTP) - SQL Injection vulnerability	🟠 High	SQL injection was detected and could allow attackers to retrieve, modify, or delete database information.

111/tcp (rpcbind) - RPC information disclosure	🟢 Low	rpcbind exposes information about available RPC services, assisting attacker reconnaissance.

139/tcp (NetBIOS/SMB) - SMB enumeration	🟡 Medium	SMB information disclosure allows attackers to gather host and share information useful for later attacks.

445/tcp (SMB) - smb-vuln-ms10-061 🟢 Not Vulnerable	NSE script returned false, indicating the system is not vulnerable to MS10-061.

445/tcp (SMB) - smb-vuln-ms10-054 🟢 Not Vulnerable	NSE script returned false, indicating the target is not affected by MS10-054.

445/tcp (SMB) - smb-vuln-regsvc-dos	⚪ Inconclusive	The script failed to complete successfully, so the vulnerability status could not be determined.

3306/tcp (MySQL) - MySQL information disclosure	🟡 Medium	Database version and service information were exposed, which may help attackers identify known exploits.

443/tcp (HTTPS) - Logjam (Weak Diffie-Hellman Parameters) 🟡 Medium	Weak Diffie-Hellman parameters reduce TLS security and could allow encrypted communications to be weakened under certain attack scenarios

**Findings:** 

The table above consolidates all of the findings reported during the Nmap --script vuln scan and categorizes each result by severity. The most significant finding was the vsFTPd 2.3.4 backdoor (CVE-2011-2523) on port 21, which was successfully exploited to obtain root-level access, making it the only Critical vulnerability confirmed through live exploitation. Several additional weaknesses were identified, including SQL injection, Telnet running without encryption, Slowloris denial-of-service susceptibility, and weak SSH cryptographic settings, all of which increase the attack surface of the target system. The SMB vulnerability checks also demonstrated that vulnerability scans can produce different outcomes: some scripts confirmed the system was not vulnerable, while others returned inconclusive results because the checks could not be completed. Overall, the scan illustrates the importance of reviewing every NSE script result individually, as findings may represent confirmed vulnerabilities, informational issues, successful mitigations, or inconclusive tests requiring additional investigation.

---

## 📊 Key Findings Summary

| Port/Service | Tool Used | Risk Level | Notes |

| 21/tcp FTP | Nmap (`vuln`, `ftp-anon`) | 🔴 Critical | vsFTPd 2.3.4 backdoor (CVE-2011-2523) — live exploitation confirmed, root access achieved |
| 23/tcp Telnet | Nmap (`vuln`) | 🟠 High | Transmits credentials in plaintext, vulnerable to interception |
| 80/tcp HTTP | Nmap (`vuln`) | 🟠 High | SQL injection vulnerability detected |
| 80/tcp HTTP | Nmap (`vuln`) | 🟡 Medium | Susceptible to Slowloris denial-of-service attack |
| 445/tcp SMB | Nmap (`-sC`, `smb-vuln*`) | 🟡 Medium | Message signing disabled; guest authentication allowed |
| 22/tcp SSH | Nmap (`vuln`) | 🟡 Medium | Weak/outdated cryptographic configuration identified |
| 3306/tcp MySQL | Nmap (`vuln`) | 🟡 Medium | Database version and service info exposed |

**Risk Levels:** 🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Low


---

## 🗺️ MITRE ATT&CK Mapping

| Action Performed | ATT&CK Tactic | Technique ID | Technique Name |
|---|---|---|---|
| Default and targeted script scanning (-sC, ftp-anon, smb-vuln*) | Reconnaissance | T1595 | Active Scanning |
| Anonymous FTP login confirmed | Initial Access | T1078 | Valid Accounts |
| vsFTPd backdoor exploitation (root shell via `id` command) | Execution | T1059 | Command and Scripting Interpreter |
| Root-level access confirmed post-exploitation | Privilege Escalation | T1068 | Exploitation for Privilege Escalation |

---

## 🛡️ Defensive Recommendations

Based on findings, the following remediations would be recommended in a real environment:

1. **vsFTPd 2.3.4 backdoor (Critical)** — Immediately upgrade or replace the FTP service; this version is known to contain an intentional backdoor and should never be used in production.
2. **Anonymous FTP login allowed** — Disable anonymous access entirely; require authenticated accounts for any FTP access.
3. **Telnet enabled (plaintext credentials)** — Disable Telnet and replace with SSH for all remote administration.
4. **SMB message signing disabled** — Enable SMB message signing to prevent tampering and man-in-the-middle attacks on SMB traffic.
5. **SQL injection on port 80** — Implement input validation/parameterized queries on the web application, and consider a WAF as an additional layer of defense.

---

## 📚 CySA+ Exam Relevance

This lab directly maps to the following CompTIA CySA+ (CS0-003) exam domains:

| Domain | Coverage |

| Security Operations (33%) | Hands-on use of Nmap and NSE scripts to perform reconnaissance and identify live services and misconfigurations |
| Vulnerability Management (30%) | Running vulnerability-check scripts (`smb-vuln*`, `vuln`), interpreting results, and cross-referencing CVE/CVSS data from NVD |
| Incident Response (20%) | Recognizing confirmed exploitation (root access via vsFTPd backdoor) as evidence of active compromise |
| Reporting & Communication (17%) | Documenting findings by severity in a structured, readable format for technical and non-technical audiences |

---

## 🔑 Technical Notes

> # Note: not all NSE vulnerability scripts behave the same way — most only detect and report (true/false), while a small number (like ftp-vsftpd-backdoor) actually attempt live exploitation. Always check script documentation or output carefully rather than assuming uniform behavior.
>
> "Always add the -n flag to Nmap scans in this VMware environment to prevent DNS resolution hangs."]
> 
> -sP and -sT contradict eachother (Can't be used together because -sP means just do a ping/host-discovery sweep, skip ports entirely but -sT tells Nmap to do a full TCP connect port scan. These commands contradict eachother.)

```bash

# Any important commands or workarounds
[-sC]
[-Pn]

# Any important commands or workarounds

# Task 1: Run default scripts (-sC) against key ports
sudo nmap -sC -Pn --disable-arp-ping -n -p 21,22,80,445 192.168.79.130

# Task 2: Check FTP for anonymous login specifically
sudo nmap --script ftp-anon -Pn --disable-arp-ping -n -p 21 192.168.79.130

# Task 3: Run all SMB vulnerability scripts against port 445
sudo nmap --script smb-vuln* -Pn --disable-arp-ping -n -p 445 192.168.79.130

# Task 4: Run the full vuln script category across all open ports
sudo nmap --script vuln -Pn --disable-arp-ping -n 192.168.79.130

# Workaround: -p requires a value directly after it — omitting the port number
# (e.g. "-p 192.168.79.130") causes Nmap to misinterpret the target IP as a
# port specification. To scan all ports instead of a specific one, remove
# the -p flag entirely rather than leaving it empty.

# Workaround: always use -n in this lab to prevent DNS resolution hangs,
# since the isolated Host-Only network has no real DNS server.

# Note: --script <name>* (wildcard) runs every NSE script matching that
# name pattern — useful for running a whole category (e.g. smb-vuln*)
# in a single command instead of specifying each script individually.
```

---

## 📌 About This Project

[1-2 sentences about how this fits into your overall portfolio and career goals.]

This repository is part of my broader cybersecurity portfolio demonstrating practical, hands-on vulnerability assessment skills from initial scanning through confirmed exploitation as I work toward a career as a cybersecurity analyst.

This repository is important because here we have built up to the point of having live proof of the actual exploitation by an automated Nmap script for the CVE-2011-2523. In a past NVD task I researched this exact vulnerability with a CVSS score of 9.8 now this has come full circle and we found out how this exact vulnerability is exploited and root access was acheived. 


**Related repositories:**
- Nmap-Host-Discovery-and-Lab-Baseline — Established a baseline of the lab network using ping sweeps and host discovery, documenting the target environment before deeper scanning began
- Nmap-Scan-Types-SYN-vs-TCP-vs-UDP — Compared SYN, TCP connect, and UDP scan types against the target, examining differences in speed, stealth, and reliability
- Service-Version-Detection-and-OS-Fingerprinting — Identified exact software versions on Metasploitable and researched real CVEs tied to them, including the vsFTPd backdoor later exploited in this repository
- Wireshark-Capture-and-Analyze-Traffic — Captured and analyzed live packet traffic, including plaintext credential exposure over Telnet
- TCPDump-CLI-Packet-Capture — Used TCPDump from the command line to capture traffic (including a live Telnet session showing plaintext credential exposure), and demonstrated saving captures to a `.pcap` file for later analysis in Wireshark

---

## 👤 Author

**Dashane James**  
Senior Field Service Technician → Cybersecurity Analyst  
📍 Yonkers, NY  
🎓 B.S. Information Technology — SUNY Canton  
🏆 CompTIA Security+ | CySA+ (In Progress)  
🔗 [GitHub](https://github.com/Dashanejames1) | [Zero Trust Cyber Security Brand](https://www.instagram.com/zerotrust_cybersecurity)

---

*This repository is part of an active portfolio demonstrating hands-on cybersecurity skills. All lab work performed in isolated environments for educational purposes.*
