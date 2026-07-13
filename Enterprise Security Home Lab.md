# Enterprise Security Home Lab - VMWare

## Objective
### This project showcases an enterprise-style cybersecurity home lab built in VMware to simulate real-world security operations. The environment integrates pfSense for network security, Wazuh with Sysmon for endpoint monitoring and threat detection, Snort for intrusion detection, and OpenVAS/Nessus for vulnerability assessments. Attack simulations using Kali Linux and Atomic Red Team were executed to emulate adversary techniques, validate custom detections, and analyze security events in Wazuh, providing hands-on experience with detection engineering, threat hunting, and incident response.

### Skills Learned
- Design and deploy home lab to simulate real-world network and security operations.
- Implement centralized security monitoring by integrating Wazuh, Sysmon, and Snort for endpoint and network visibility.
- Emulate adversary techniques using Atomic Red Team and Kali Linux to validate detections and strengthen security monitoring.
- Perform vulnerability assessments with OpenVAS and Nessus to identify and remediate security weaknesses.
- Develop hands-on skills in threat detection, log analysis, network segmentation, and firewall management.

### Tools Used
- Virtualization: VMware
- Network Security: pfSense, Snort, Wireshark
- SIEM/XDR: Wazuh, Sysmon
- Adversary Emulation: Atomic Red Team, Kali Linux
- Vulnerability Management: Nessus, OpenVAS
- Threat Intelligence & Patching: VirusTotal, Action1

## Steps

Ref 1: Network Diagram

<img width="800" height="500" alt="Screenshot 2026-07-12 170302" src="https://github.com/user-attachments/assets/9fc4333d-1cd6-4c16-a044-cf6a53ca4abb" />

 
This diagram shows the segmented network architecture of my lab. The environment uses pfSense as the central router and firewall to separate the internal network from vulnerable systems. Wazuh provides centralized security monitoring and endpoint telemetry, while Kali Linux is used to simulate 
attacks against Windows and Metasploitable systems for threat detection, vulnerability assessment, and security testing.

---------------------------
Ref 2: Network Intrusion Detection with Snort

<img width="689" height="305" alt="Screenshot 2026-07-12 162228" src="https://github.com/user-attachments/assets/6151049c-4bbe-4d83-87cc-5c973cd30e32" />

<img width="664" height="482" alt="Screenshot 2026-07-12 162601" src="https://github.com/user-attachments/assets/99383a09-f5a8-4206-87e3-391942fbf480" />

Snort IDS/IPS was deployed on pfSense to monitor network traffic.

Custom detection rules were created to identify:

- ICMP traffic
- SSH connections
- Nmap reconnaissance activity
- HTTP probe activity
  
Nmap scans were launched against Metasploitable systems to validate the custom detection rules.

---------------------------
Ref 3: Vulnerability Assessment

<img width="625" height="299" alt="Screenshot 2026-07-12 162928" src="https://github.com/user-attachments/assets/730df36a-cfb5-46dc-8e8f-eb663fd13a6c" />

<img width="685" height="147" alt="Screenshot 2026-07-12 163056" src="https://github.com/user-attachments/assets/0ada8408-e698-4d08-8ffe-ae033743d06c" />

OpenVAS and Nessus were used to conduct vulnerability assessments against systems within the vulnerable network.

An OpenVAS scan against Metasploitable 2 identified:

- 13 Critical vulnerabilities
- 11 High severity vulnerabilities
- 40 Medium severity vulnerabilities
- 6 Low severity vulnerabilities
- 20 exposed network ports

Common exposed services included HTTP, FTP, Telnet, SSH, and SMTP.

Nessus was also used to validate vulnerability findings and identified critical security issues. In the result of a full scan of the 172.30.1.0/24 network nessus shows us outdated operating systems, backdoor detection, and weak VNC authentication.

---------------------------
Ref 4: SIEM & Endpoint Monitoring

<img width="709" height="268" alt="Screenshot 2026-07-12 164011" src="https://github.com/user-attachments/assets/9e82beed-9c3f-45e2-95db-6956f32e991d" />

Wazuh SIEM/XDR was deployed to provide centralized security monitoring across Windows and Linux endpoints.

Sysmon logs were integrated with Wazuh to provide enhanced endpoint telemetry and visibility into:

- Process execution
- Process injection
- DLL activity
- File creation and deletion
- Network connections
- Suspicious system activity

---------------------------
Ref 5: Vulnerability Remediation

<img width="623" height="352" alt="Screenshot 2026-07-12 164308" src="https://github.com/user-attachments/assets/4eb7bc43-bd65-4b59-b21e-f8644bc48641" />

<img width="677" height="161" alt="Screenshot 2026-07-12 164317" src="https://github.com/user-attachments/assets/32ca0100-83b2-4497-ac00-aac661ea59d3" />

Action1 was deployed for endpoint and patch management. This was used to identify outdated applications and automate software updates.

Successfully remediated outdated versions of:

- Google Chrome
- Git

Google Chrome contained a critical vulnerability with a CVSS score of **9.6**.

Both applications were updated to supported versions to mitigate the identified security risks.



