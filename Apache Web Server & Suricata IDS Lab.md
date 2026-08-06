# Apache Web Server & Suricata Network Intrusion Detection Lab

## Objective
### The objective of this lab is to deploy an Apache web server and configure Suricata as a network intrusion detection system (IDS) to monitor and analyze web traffic for suspicious or malicious activity.

### Skills Learned
- Installed, configured, and managed an Apache web server on Ubuntu
- Configured Suricata to monitor network traffic and implemented detection rules
- Analyzed HTTP traffic and generated reconnaissance activity using Nmap
- Investigated Apache access.log, Suricata fast.log and eve.json telemetry
- Created and tested custom Suricata rules to identify suspicious web activity.

### Tools Used
- Ubuntu Linux
- Apache web server
- Suricata IDS/IPS
- Nmap

## Steps

1: Apache Web Server Installation

<img width="1177" height="440" alt="Screenshot 2026-07-31 235945" src="https://github.com/user-attachments/assets/0af56a76-8a97-42de-932a-9c2e0bf3bc24" />
<img width="1205" height="762" alt="Screenshot 2026-08-01 001500" src="https://github.com/user-attachments/assets/067ce9ed-9c39-4cbc-b8d3-69745a7038c1" />

Verifying status of Apache:
 - Used the command "sudo systemctl status apache2"  with root privileges to verify if the web server is running. Also verified in the web browser
 - The IP address of the server is 192.168.1.127

---

2: Generating Traffic To Apache

<img width="2397" height="741" alt="Screenshot 2026-08-02 171150" src="https://github.com/user-attachments/assets/b90b3bc9-199a-41f4-9bda-2ecaec679b67" />

- Generating test traffic from Kali and viewing the connection from the "access.log" file. This file logs any connections that are made to the web server
- Used the command "curl http://192.168.1.127" from Kali to send successful GET requests to Apache

---

3: Suricata Installation and Configuration

<img width="1207" height="510" alt="Screenshot 2026-08-01 151212" src="https://github.com/user-attachments/assets/edde7baa-3806-49c5-9d26-ad277e113280" />

- Successfully installed and confirmed Suricata is loaded, enabled, and actively running it's latest version using the config at /etc/suricata/suricata.yaml

<img width="1038" height="430" alt="Screenshot 2026-08-01 152107" src="https://github.com/user-attachments/assets/d4c14446-1abe-4fa7-bab5-5cc3ddd1c613" />

- Defining HOME_NET as 192.168.1.0/24, telling Suricata which addresses represent the internal/local network for alerting purposes
- Set EXTERNAL_NET as !$HOME_NET, meaning anything outside the home network is treated as external

<img width="1038" height="158" alt="Screenshot 2026-08-01 151817" src="https://github.com/user-attachments/assets/75f9248b-dff8-4e86-b705-139f10315bb0" />

- Configuring Suricata to detect traffic and listen on the enp0s3 interface
- This will detect any traffic to 192.168.1.127 which is the Apache web server

---

4: Suricata Rule Creation

<img width="1165" height="225" alt="Screenshot 2026-08-03 194555" src="https://github.com/user-attachments/assets/fc90e91d-d14f-4d75-9309-ede209e2ca9f" />

- Created a custom rule file name "local.rules" to alert on ICMP pings and TCP SYN connection attempts to SSH (22), FTP (21), and HTTP (80) on the web server

---

5. Testing Suricata and Apache Alert Detection

<img width="2386" height="745" alt="Screenshot 2026-08-04 160427" src="https://github.com/user-attachments/assets/87be1bd4-8874-49f5-a44e-86e4957c012b" />

- Ran a full port scan (nmap -p 1-1000 192.168.1.127) from Kali, confirming only SSH (22) and HTTP (80) were open


<img width="1202" height="113" alt="Screenshot 2026-08-04 160713" src="https://github.com/user-attachments/assets/09adc573-d5ca-4cc4-8d73-5eb128eead03" />

- Using "grep" to filter log output to isolate just the HTTP related alerts from the noisier full log


<img width="2266" height="419" alt="Screenshot 2026-08-04 161753" src="https://github.com/user-attachments/assets/b6e84bed-405b-470b-8af6-7e822911046a" />

- Ran "gobuster dir" against the web server using the "dirb/common.txt" wordlist, enumerating directories and returning "index.html" (200) along with several 403-protected paths
- Correlated the scan in Apache's access.log, filtering for "password" related requests to show gobuster systematically probing for password-reset/change endpoints





