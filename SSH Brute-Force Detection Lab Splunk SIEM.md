#  SSH Brute-Force Detection Lab — Splunk 
## Objective
### Built an end-to-end security monitoring lab simulating SSH brute-force attacks and detecting them in real time using Splunk. Configured a Ubuntu target machine, generated realistic attack traffic, built SPL queries and automated alerts to identify threats.

### Skills Learned
- Developed Splunk SIEM monitoring and SPL query skills to analyze SSH authentication activity.
- Gained hands-on experience simulating and detecting SSH brute-force attacks.
- Analyzed failed and successful authentication logs to identify suspicious login patterns.
- Created automated Splunk alerts to detect potential brute-force activity in real time.
- Improved threat detection and log analysis skills by investigating source IPs and unauthorized connection attempts.

### Tools Used
- Splunk
- SSH
- Linux Authentication Logs
- VMware

## Steps

Ref 1: Simulating SSH Authentication Attempts

<img width="678" height="333" alt="Screenshot 2026-07-14 072955" src="https://github.com/user-attachments/assets/4e2f7225-3ef2-4725-8ba6-0cb2255517e7" />

- Simulated brute-force SSH attacks from a Windows host onto a Linux VM and captured authentication logs for monitoring and analysis

----

Ref 2: Detecting Unauthenticated SSH Connections

<img width="844" height="136" alt="Screenshot 2026-07-14 073858" src="https://github.com/user-attachments/assets/11b07da0-c4ff-40f0-8a16-0b8de9042d5d" />

- Used SPL queries in Splunk to identify connection attempts without successful authentication and visualize activity by source IP over time.
- Querying event_type="Connection Without Authentication" visualized as a timechart by source IP. Returns 286 events across multiple 10.0.0.x hosts

----

Ref 3: Creating a Brute-Force Detection Alert

<img width="650" height="580" alt="Screenshot 2026-07-14 074332" src="https://github.com/user-attachments/assets/1478da5a-cf8c-43db-a072-eb88b9593225" />

- Configured a scheduled Splunk alert to automatically detect repeated authentication activity and trigger when defined detection conditions are met.
- It's set to run weekly, trigger when results are greater than 0, fire once per result, and add entries to Triggered Alerts at Medium severity.

----

Ref 4: Analyzing Failed and Successful SSH Attempts

<img width="844" height="311" alt="Screenshot 2026-07-14 074613" src="https://github.com/user-attachments/assets/e7cae4a3-2856-4509-a657-79c023837e75" />

- More failed SSH authentication events 

<img width="843" height="327" alt="Screenshot 2026-07-14 075239" src="https://github.com/user-attachments/assets/e424ae89-094b-450a-a830-c2afca3fade4" />

- Queried for successful SSH authentication events to distinguish legitimate logins from failed attempts

