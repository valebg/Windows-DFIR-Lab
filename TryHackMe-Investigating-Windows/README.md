# 🛡️ TryHackMe: Investigating Windows — Incident Response Writeup

Detailed forensic analysis and investigation writeup for the **Investigating Windows** laboratory on TryHackMe.

---

## 📌 Executive Summary

A Windows Server system was compromised by an external adversary. This writeup documents the incident response process, artifact extraction, and threat hunting methodology used to analyze the host, reconstruct the attack timeline, identify persistence mechanisms, and extract all relevant Indicators of Compromise (IOCs).

---

## 📊 Summary of Forensic Artifacts & IOCs

| Forensic Question / Investigation Item | Finding / Answer | Source & Evidence Location |
| :--- | :--- | :--- |
| **Windows OS Version & Year** | `Windows Server 2016` | System Information (`systeminfo` / `winver`) |
| **Last Logged-In User** | `Administrator` | Security Event Logs / User Profiles |
| **John's Last Logon Timestamp** | `03/02/2019 5:48:32 PM` | `net user John` / User Artifacts |
| **Startup Connection IP** | `10.34.2.3` | Registry Run key (`HKLM\...\Run`) |
| **Admin Accounts (Excl. Administrator)** | `Guest, Jenny` | Local Group Enumeration (`net localgroup Administrators`) |
| **Malicious Scheduled Task Name** | `Clean File system` | Task Scheduler Library (`taskschd.msc`) |
| **Daily Executable Targeted by Task** | `nc.ps1` | Task Action Properties (`C:\TMP\nc.ps1`) |
| **Local Port Listening via Script** | `1348` | Code inspection of `nc.ps1` (`-l -p 1348`) |
| **Jenny's Last Logon** | `Never` | `net user Jenny` output |
| **Date of Initial Compromise** | `03/02/2019` | Security Event Log (Event ID `4672`) |
| **First Special Privileges Assignment** | `03/02/2019 04:04:49 PM` | First timestamp of Event ID `4672` |
| **Credential Dumping Tool Used** | `Mimikatz` | Binaries located in `C:\TMP` |
| **External C2 Server IP (DNS Poisoning)**| `76.32.97.132` | Poisoned mapping in `C:\Windows\System32\drivers\etc\hosts` |
| **Webshell File Extension** | `jsp` | Web root inspection (`C:\inetpub\wwwroot\`) |
| **Last Open Firewall Port** | `1337` | Inbound Rules in Windows Defender Firewall (`wf.msc`) |
| **Targeted Site for DNS Poisoning** | `google.com` | `hosts` file entry (`76.32.97.132 google.com`) |

---

## 🔍 Detailed Forensic Walkthrough

### 1. User & Account Auditing
- **Account Privilege Enumeration:** Executed `net localgroup Administrators` in CMD to identify local accounts with administrative privileges besides the default Administrator. Accounts found: `Guest` and `Jenny` (sorted alphabetically as `Guest, Jenny`).
- **Logon History Verification:** Executed `net user Jenny` to verify logon history, confirming `Last logon: Never`.

### 2. Persistence & Auto-Start Analysis
- **Registry Auto-Run Inspection:** Checked `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` for suspicious registry entries attempting outbound connections upon boot, discovering local network initialization to `10.34.2.3`.
- **Scheduled Task Audit:** Inspected Task Scheduler Library (`taskschd.msc`) and discovered a suspicious task named `Clean File system`.
- **Trigger:** Configured to run daily at 4:00 PM.
- **Action:** Executing a PowerShell Netcat wrapper script at `C:\TMP\nc.ps1`.
- **Code Analysis:** Inspected `C:\TMP\nc.ps1` via Notepad/PowerShell, confirming it binds locally to port `1348`.

### 3. Event Log & Timeline Analysis
- **Initial Breach Timestamp:** Filtered Windows Security Logs in Event Viewer (`eventvwr.msc`) for **Event ID 4672** (*Special privileges assigned to new logon*).
- **Results:** Identified the compromise date as **`03/02/2019`** with elevated privilege delegation starting at **`03/02/2019 04:04:49 PM`**.

### 4. Malware & Tooling Identification
- **Credential Harvesting:** Found `mimikatz.exe` inside `C:\TMP`, indicating active credential dumping.
- **Webshell Artifacts:** Audited IIS Web Server directory `C:\inetpub\wwwroot\` and identified an uploaded `.jsp` webshell utilized for remote code execution.
- **Firewall Modifications:** Opened Windows Defender Firewall with Advanced Security (`wf.msc`) and inspected **Inbound Rules**. Found a newly created rule allowing incoming connections on port **`1337`**.

### 5. Network & DNS Poisoning Investigation
- **Hosts File Inspection:** Checked `C:\Windows\System32\drivers\etc\hosts` for local DNS hijacking.
- **Findings:** Discovered an entry overriding `google.com` to redirect traffic to the adversary's Command & Control (C2) IP address:
```text
76.32.97.132   google.com
🛠️ Investigation Tools & Commands Used
DOS
:: Account Auditing
net localgroup Administrators
net user Jenny
net user John

:: Auto-Start & Registry Queries
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

:: Network & Firewall Verification
type C:\Windows\System32\drivers\etc\hosts
wf.msc
PowerShell
:: Scheduled Task & Process Inspection
Get-ScheduledTask | Where-Object {$_.State -ne "Disabled"}
Get-Content C:\TMP\nc.ps1

:: Event Log Filtering for Special Privileges
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4672} | Select-Object -First 5
🎯 Conclusion & Remediation Steps
Host Isolation: Immediately disconnect the endpoint from the network to prevent further C2 communication.

Account Cleanup: Disable unauthorized administrative access for Guest and revoke compromise-related accounts.

Persistence Removal: Delete the malicious scheduled task Clean File system and remove C:\TMP\nc.ps1.

Firewall & Network Cleanup: Delete the inbound rule for port 1337 and restore the default hosts file.

Credential Reset: Enforce domain-wide password resets due to Mimikatz execution on the endpoint.
