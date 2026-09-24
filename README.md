# Generating-Attacks-and-Defending-in-Splunk


**Author:** Dashane James  
**Lab Environment:** [e.g. VMware Workstation | Kali Linux | Metasploitable 2]  
**Purpose:** [The goal of this repository is to use Nmap scripts for targeted vulnerability checks, bridging network scanning and vulnerability assesment.]
**Status:** 🔵 Completed

---

## 📋 Overview

[This repository documents combining the network scanning of Nmap with the task of vulnerabIlity assessments. This creates a simpler way to both scan a network and identify vulnerabilities simultaneously which is very useful for a cybersecurity analyst or penetration tester.]

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


# Command used
[sudo tcpdump -i eth0 -w ~/lab_capture.pcap] -tcp dump

<img width="320" height="184" alt="image" src="https://github.com/user-attachments/assets/4e6c8a16-8562-422e-9b5f-3c80357e89e5" />


[sudo nmap -sT -Pn --disable-arp-ping -n 192.168.79.130] - nmap

<img width="346" height="312" alt="image" src="https://github.com/user-attachments/assets/b5a7ba69-4468-438e-a7fd-9fbc2f266900" />

# Output

For this task I opened two terminal windows simultaneously. In the first terminal I started a TCPDump capture using `sudo tcpdump -i eth0 -w ~/lab_capture.pcap`, which began listening on the eth0 network interface and saving all captured packets to a file called lab_capture.pcap. In the second terminal I ran an Nmap scan against Metasploitable (192.168.79.130) while TCPDump was actively capturing in the background. The Nmap scan returned a full list of open ports and services on the target which confirmed the target was live and its attack surface was fully exposed. Once the Nmap scan completed I stopped the TCPDump capture with Ctrl+C, which saved all the network traffic generated during the scan to lab_capture.pcap. This file contains the raw packet-level evidence of the port scan and will be imported into Splunk for analysis. This simulates how a SOC analyst would capture and investigate suspicious network reconnaissance activity.




### 2. [Import TCPDump capture into Splunk]


# Command used
[sudo nmap --script ftp-anon -Pn --disable-arp-ping -n -p 21 192.168.79.130]


# Output




### 3. []
[] 


# Command used
[]

# Output




### 4. []


#Command
[sudo nmap --script vuln -Pn --disable-arp-ping -n 192.168.79.130]

# Output



**Findings:** 
The Vuln script category confirmed and actively exploited the vsftpd 2.3.4 backdoor (CVE-2011-2523) on port 21, achieving root-level command execution. The script ran the shell command id through the backdoor and received uid=0(root) gid=0(root), which is definitive proof of successful remote access. This was the strongest result of any scan in this project, since it moved beyond detection into actual proof of compromise. Notably, this was one of the only scripts in the scan that performed live exploitation. Most other scripts in the "vuln" category only detect and report whether a vulnerability exists, without attempting to exploit it. However, port 21's script is a special case because the vsftpd 2.3.4 backdoor is a well documented, simple command-injection vulnerability that Nmap's script can reliably trigger and verify with a single command. Most other vulnerability scripts test for more complex or riskier conditions, like crashing a service, so they are written to stop at detection rather than attempt exploitation.

### 5. [Document every vuln NSE finds with its severity.]
For this task I continued to review the vulnerability output that I received in task 4. However, this time I reviewed and  documented each vulnerability that was displayed. I then categorized them by port, service, vulnerability, NSE Script, severity, along with any other notes or important details.


### Output

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
