# Sigma Rules Detection Lab

## Objective
### This lab was completed on TryHackMe which simulated an incident where there was unusual activity across several machines on a network. Two suspicious behaviors were flagged: an unknown entity creating scheduled tasks on a host and ransomware activity on the same environment. My task was to act as the analyst, building Sigma detection rules for both activities, translating them into ElasticSearch/Lucene queries using Uncoder.IO, and running those queries in Kibana to confirm the malicious activity and pull out key forensic details (task name, run time, dropped file name, event code, and ransom note contents).

### Skills Learned
- Writing Sigma rules from scratch for both process creation and file creation log sources.
- Translating Sigma rules into platform-specific queries (ElasticSearch/Lucene) using Uncoder.IO.
- Querying and pivoting through logs in Kibana to validate detection logic against real telemetry.
- Interpreting Sysmon event data (Event ID 1 - Process Creation, Event ID 11 - File Creation) to reconstruct attacker behavior.
- Tracing a command execution chain from a scheduled task through to a dropped ransom note file.

### Tools Used
- Elastic/Kibana: for searching, filtering, and inspecting ingested Sysmon logs.
- Sigma: for writing vendor-agnostic detection rules.
- Uncoder.IO: for converting Sigma rules into ElasticSearch/Lucene queries.
- Sysmon: as the log source providing process creation and file creation telemetry.


## Steps

1: Building the Ransomware Detection Sigma Rule


<img width="796" height="338" alt="Sigma rule for ransomware detection" src="https://github.com/user-attachments/assets/96e45f22-a6ad-4ead-9452-38acdaaaa583" />

 
- Using the details from the process creation event, I wrote a Sigma rule targeting the file_creation log source. The rule looks for any file creation performed by cmd.exe where the resulting filename ends in .txt

---------------------------
2: Investigating the Ransomware Activity

Ref: Uncoder.IO translating the Sigma rule from Sigma format into an ElastAlert
<img width="2014" height="472" alt="Uncoder rule to query conversion" src="https://github.com/user-attachments/assets/35dc8bdb-f80c-443e-8935-fc8fa1b92758" />

Ref: Kibana results for the translated query, showing the File Created event (Event ID 11)
<img width="2267" height="701" alt="File creation " src="https://github.com/user-attachments/assets/7385ba1a-45a9-4f3d-a09c-95430ac7bfb5" />

- I used Uncoder.IO to translate the Sigma rule into an ElasticAlert/Lucene query, then ran the generated query in Kibana. The query (process.executable.text:"cmd.exe" AND file.path.text:*.txt) returned a single matching event - Sysmon Event ID 11 (File Created) - confirming cmd.exe created YOUR_FILES.txt on the Administrator's desktop at the same timestamp as the process creation event.


---------------------------
3: Building the Scheduled Task Detection Sigma Rule

<img width="1019" height="404" alt="Sigma rule schtasks" src="https://github.com/user-attachments/assets/ac068e8f-7d2b-4705-a52c-23fb13b35f13" />

- I then wrote a second Sigma rule under the process_creation log source, detecting any execution of schtasks.exe whose command line contains both schtasks and create. I added a filter to exclude events where the user is NT AUTHORITY\SYSTEM, so the rule focuses on non-system accounts creating scheduled tasks 
---------------------------
4: Investigating the Scheduled Task Activity

Ref: Translating and Validating the Scheduled Task Rule
<img width="1878" height="529" alt="Screenshot 2026-07-19 011856" src="https://github.com/user-attachments/assets/9076b744-4759-43a7-bb21-abf2192edac6" />

Ref: Kibana results showing the schtasks.exe process creation event
<img width="2265" height="800" alt="Schtask exe creation" src="https://github.com/user-attachments/assets/df7ce50a-9ec9-4151-8b6c-b5acdfddcb69" />

- Next, I pivoted to the scheduled task logs. I searched Kibana using the query from Uncoder.IO - (process.executable.txt:"schtasks".exe AND (process.command_line.txt:"*schtasks*" AND process.command_line:*create*)) AND NOT (User:NT AUTHORITY\SYSTEM) - This surfaced a process creation event showing schtasks.exe being run with /Create /SC ONCE /TN spawn /TR C:\windows\system32\cmd.exe /ST 20:10, spawned as a child of cmd.exe, revealing a scheduled task named spawn set to run at 20:10.


---------------------------
5:  Summarizing the Findings


<img width="2247" height="722" alt="Content of text file" src="https://github.com/user-attachments/assets/da8c1a56-7e3b-49d8-844d-213643ee3ab9" />

Piecing both detections together, the attack chain was:
- An unknown entity used schtasks.exe to create a scheduled task named spawn, set to trigger cmd.exe at 20:10.
- The cmd.exe execution was then used to drop a ransom note file, YOUR_FILES.txt, on the Administrator's desktop, containing the text "T1486 - Purelocker Ransom Note."
- Both events were captured via Sysmon Event ID 1 and Event ID 11 and both were successfully detected using the custom Sigma rules built during this lab.




