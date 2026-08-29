# TryHackMe: Conti Ransomware & ProxyShell Incident Investigation

## Overview
This repository contains my investigation write-up for the **Conti** room on TryHackMe. The room focuses on investigating a real-world compromised Windows environment using **Splunk (SIEM)** and **Sysmon logs**. 

The threat actor exploited Microsoft Exchange (ProxyShell vulnerability chain) to drop a web shell, gain initial access, dump system hashes, migrate processes, and deploy **Conti Ransomware**.

---

## Incident Timeline & Analysis

### 1. Initial Access & Exploitation (ProxyShell)
The attacker leveraged a chain of three Microsoft Exchange vulnerabilities (**ProxyShell**) to bypass authentication, elevate privileges, and write arbitrary files:
* **CVE-2021-31207** (Post-auth Arbitrary File Write)
* **CVE-2021-34473** (Pre-auth Path Confusion / SSRF)
* **CVE-2021-34523** (Elevation of Privilege)

### 2. Persistence & Web Shell Deployment
* **Web Shell Filename:** `i3gfPctK1c2x.aspx`
* **Path:** `C:\Program Files\Microsoft\Exchange Server\V15\FrontEnd\HttpProxy\owa\auth\`
* **Defense Evasion:** Executed `attrib.exe -r ...` via `cmd.exe` to modify file attributes and remove read-only restrictions.

### 3. Execution, Account Creation & Credentials Access
* **New Account Creation:** Attacker created a backdoor user using `net user` / `net localgroup administrators`.
* **Credential Dumping:** Dumped LSASS memory to extract NT/LM hashes:
* **Process Image:** `C:\Windows\System32\lsass.exe`
* **Process Migration:** Migrated processes for persistence (from initial breach process to legitimate system processes).

### 4. Malware Impact (Conti Ransomware)
* **Ransomware Execution & MD5 Hash:** Extracted via Sysmon `EventID 1` (Process Creation) / `EventID 11` (File Create).
* **Mass File Modifications:** Detected batch drop of ransom notes (`readme.txt`) across multiple folder directories.

---

## Key Artifacts & MITRE ATT&CK Mapping
* **T1190** — Exploit Public-Facing Application (ProxyShell)
* **T1505.003** — Server Software Component: Web Shell (`i3gfPctK1c2x.aspx`)
* **T1003.001** — OS Credential Dumping: LSASS Memory (`lsass.exe`)
* **T1222.001** — File and Directory Permissions Modification (`attrib.exe -r`)
* **T1486** — Data Encrypted for Impact (Conti Ransomware)

---

## Tools Used
* **Splunk Enterprise** (SPL queries for IIS, Sysmon EventCodes 1, 10, 11)
* **Sysmon Log Analysis**
