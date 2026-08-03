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

1: Network Topology

<img width="1192" height="490" alt="Screenshot 2026-07-26 154208" src="https://github.com/user-attachments/assets/2af71be6-fda2-497b-b4d0-eb00a7b6e493" />

Built the topology in Cisco Packet Tracer with Router1 segmenting two internal subnets:
 - 10.1.1.0/24 - HTTP Server1 (10.1.1.100) and HTTP Server2 (10.1.1.101), behind Switch1 
 - 10.1.2.0/24 - Inside PC1 (10.1.2.101) and Inside PC2 (10.1.2.102), behind Multilayer Switch0

---

2: Created extended ACL on Router1

<img width="550" height="102" alt="Screenshot 2026-07-26 152529" src="https://github.com/user-attachments/assets/0de8ea01-0721-49ad-8d4b-675d735b44b2" />

- Line 10: Inside PC1 (10.1.2.101) can reach HTTP Server1 (10.1.1.100) on HTTP only
- Line 20: Inside PC2 (10.1.2.102) can reach HTTP Server2 (10.1.1.101) on HTTPS only
- Line 30: Explicit deny for all other traffic from 10.1.2.0/24 to 10.1.1.0/24
- Line 40: All other outbound traffic from 10.1.2.0/24 permitted

---

3: Verified the configuration with test cases

<img width="845" height="530" alt="Screenshot 2026-07-26 153542" src="https://github.com/user-attachments/assets/95c8546a-9ea4-4400-8c6a-0b69b57dce11" />

- Inside PC1 successfully loaded "http://10.1.1.100" in its browser, confirming HTTP access to HTTP Server1


<img width="781" height="437" alt="Screenshot 2026-07-26 153623" src="https://github.com/user-attachments/assets/6f975d34-93cc-4810-8713-a62497074925" />

- Inside PC2 successfully loaded "https://10.1.1.101" in its browser, confirming HTTPS access to HTTP Server2


<img width="577" height="97" alt="Screenshot 2026-07-26 153721" src="https://github.com/user-attachments/assets/e74a07e8-0a62-4a49-90cd-616798c3088c" />

Confirmed via "show access-lists" that hit counters incremented as expected: 
 - 6 matches on the PC1 -> Server1 HTTP rule
 - 6 matches on the PC2 -> Server2 HTTPS rule
 - 30 matches on the deny rule
