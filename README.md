# Windows DFIR Lab

## Overview

This project documents a practical Windows Digital Forensics and Incident Response (DFIR) laboratory built for SOC training.

The lab connects a Windows workstation with a Parrot Security OS analysis workstation. Windows generates native Event Logs and additional Sysmon telemetry, while Parrot is used to collect and inspect the resulting evidence through SMB.

The project focuses on understanding authentication events, process execution, parent-child process relationships, Sysmon telemetry, and remote SMB authentication from an analyst's perspective.

---

## Lab Environment

| Component | Description |
|---|---|
| Endpoint | Windows workstation |
| Analysis Workstation | Parrot Security OS |
| File Sharing | SMB / Samba |
| Native Logs | Security, System, Application |
| Additional Telemetry | Microsoft Sysmon |
| Sysmon Channel | `Microsoft-Windows-Sysmon/Operational` |
| DFIR Share | `DFIR_Logs` |
| Lab Account | `soclab` |

> Hostnames, usernames, and IP addresses used in the laboratory should be treated as lab-specific identifiers and sanitized before public reuse.

---

## Lab Architecture

```text
Windows Endpoint
+-------------------------+
| Windows Event Logging   |
|                         |
| Security / System /     |
| Application             |
|                         |
| Sysmon                  |
|   └─ Process telemetry  |
|   └─ Parent/child chain |
|   └─ Command line       |
|   └─ SHA256             |
+------------+------------+
|
| SMB
| DFIR_Logs
|
+------------v------------+
| Parrot Security OS      |
|                         |
| smbclient               |
| EVTX collection         |
| grep / awk / less       |
| DFIR analysis           |
+-------------------------+
ObjectivesBuild a controlled Windows DFIR/SOC training environment.Collect native Windows Event Logs.Deploy and verify Microsoft Sysmon.Understand the difference between native Windows logging and Sysmon telemetry.Investigate successful and failed authentication events.Analyze Windows Logon Type 2 and Logon Type 3.Trace process creation and parent-child relationships.Collect Windows logs remotely from Parrot OS through SMB.Investigate SMB authentication failures and successful access.Prepare evidence for further DFIR tooling and SIEM integration.Windows Event LogsThe laboratory uses the standard Windows event channels:Security — authentication, logon/logoff, privileges, account activity.System — operating-system and service-related events.Application — application-level events.Important Security Event IDs examined during the lab include:Event IDPurpose4624Successful account logon4625Failed account logon4672Special privileges assigned to a new logon4799A security-enabled local group membership was enumeratedLogon TypesA key distinction investigated in the lab was the Windows Logon Type:TypeMeaning2Interactive logon — local/console session3Network logon — access over the network, such as SMBThis distinction was especially useful when investigating access from Parrot OS to the Windows workstation.SysmonMicrosoft Sysmon was installed as an additional endpoint telemetry source.It writes its own events to:PlaintextMicrosoft-Windows-Sysmon/Operational
Sysmon does not simply reread the Security log and display it with more detail. It generates additional telemetry and stores those events in its own event channel.The laboratory verified Sysmon with:PowerShellGet-Service Sysmon64

Get-WinEvent -ListLog * |
Where-Object LogName -like "*Sysmon*"
The Sysmon log was observed with a circular maximum size of approximately 64 MiB.Sysmon Event ID 1 — Process CreationThe laboratory examined fields such as:ImageCommandLineUserLogonIdTerminalSessionIdIntegrityLevelProcessIdProcessGuidParentProcessIdParentImageParentCommandLineParentUserSHA256This allows an analyst to move beyond:"A process executed."and ask:Who executed it, what was executed, what command line was used, and which parent process created it?Example:PowerShellGet-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10 |
Format-List TimeCreated, Id, Message
Process InvestigationSysmon provided practical visibility into process chains.For example:Plaintextsvchost.exe
└── taskhostw.exe
└── SoftLandingTask.exe
The telemetry also showed the associated user, logon session, integrity level, command line and SHA256 hash.This is important for SOC analysis because the executable name alone is rarely sufficient. The parent process, command line, user context and execution location provide additional context.SMB Authentication InvestigationThe lab also simulated remote SMB authentication from Parrot Security OS.Failed authentication attempts were visible in Windows Security Event 4625.One observed failure showed:PlaintextLogon Type: 3
Account Name: valera
Account Domain: ANA

Workstation Name: PARROT
Source Network Address: [lab IP]

Authentication Package: NTLM
The failure codes distinguished different authentication problems:Plaintext0xC000006D — authentication failure
0xC000006A — incorrect password
0xC0000064 — unknown/incorrect username
The important DFIR observation was that the Windows endpoint recorded:PlaintextPARROT → Windows
together with the requested account and authentication result.After the correct laboratory credentials were used, SMB access to the DFIR_Logs share was successful.Evidence CollectionThe Windows workstation exposed a dedicated SMB share for laboratory evidence:PlaintextDFIR_Logs
The share contained exported event logs such as:PlaintextSecurity.evtx
System.evtx
Application.evtx
and additional laboratory files.Parrot accessed the share using smbclient.Example workflow:Bashsmbclient -L //WINDOWS_HOST -U USERNAME
Then:Bashsmbclient //WINDOWS_HOST/DFIR_Logs -U USERNAME
Inside the SMB session:Plaintextls
The purpose is to separate evidence collection from analysis: Windows remains the evidence source, while Parrot acts as the analysis workstation.Investigation WorkflowThe laboratory follows a simplified SOC/DFIR workflow:PlaintextEvent
|
v
Identify Event ID
|
v
Identify User / Account
|
v
Identify Logon Type
|
v
Identify Source / Network Information
|
v
Identify Process
|
v
Trace Parent Process
|
v
Check Command Line / Hash
|
v
Correlate with Other Events
|
v
Determine Normal vs Suspicious
The main lesson is:An Event ID by itself is only one piece of evidence.An analyst should correlate who, what, when, where, and how.🔬 TryHackMe Lab: Investigating Windows (Case Study)As part of this laboratory framework, a practical digital forensics investigation was conducted on a compromised Windows Server machine from TryHackMe.Key Forensic Findings & IOCsQuestion / ArtifactFindings & EvidenceInvestigation SourceCompromise Date03/02/2019Windows Security Log (Event 4672)First Special Privileges Assigned03/02/2019 04:04:49 PMSecurity Event Viewer timestampMalicious Scheduled TaskClean File systemTask Scheduler (taskschd.msc)Daily Malicious Filenc.ps1Task Action (C:\TMP\nc.ps1)Local Listening Port1348Script parameter inspectionLast Firewall Port Opened1337Windows Defender Firewall (wf.msc)Password Dumping ToolMimikatzBinaries in C:\TMPUploaded Webshell ExtensionjspIIS Web Root C:\inetpub\wwwrootDNS Poisoning Targetgoogle.comHosts File C:\Windows\System32\drivers\etc\hostsCommand & Control (C2) IP76.32.97.132Mapped IP address in hosts filePractical Skills DemonstratedWindowsEvent ViewerPowerShellWindows Security Event LogsSysmonProcess investigationAuthentication investigationSMB configurationParrot Security OSsmbclientgrepsortuniqlessEVTX collectionCommand-line investigationDFIR / SOC ConceptsAuthentication analysisLogon Type analysisProcess lineageParent-child relationshipsCommand-line investigationHash-based evidenceNetwork-origin analysisEvidence collectionLog correlationLessons LearnedNative Windows Event Logs and Sysmon provide different types of telemetry.Sysmon maintains its own event channel rather than replacing the native Security log.Logon Type 3 is particularly useful when investigating network-based authentication such as SMB.A failed authentication event can reveal the attempted username, source workstation, source address and authentication method.Sysmon Event ID 1 provides significantly more process context than a simple process name.Parent-child relationships are valuable when determining why a process appeared.A dedicated DFIR SMB share provides a simple way to move evidence from a Windows endpoint to a Linux analysis workstation.Circular event-log storage limits disk consumption but means older events can eventually be overwritten.The same investigation concepts can later be applied to centralized SIEM/EDR environments.Next StepsThe laboratory can be extended with:EVTX parsing and timeline creationSigma rulesChainsawHayabusaMITRE ATT&CK mappingDetection engineeringSysmon configuration tuningSIEM ingestionEDR telemetry correlationActive Directory investigationCompleted TryHackMe: Attacktive Directory room covering User Enumeration, AS-REP Roasting, DCSync, and Pass-the-Hash.Detailed writeup available in: ./Active-Directory-Attacktive-Directory/README.mdNetwork traffic analysis with WiresharkRepository StructurePlaintextWindows-DFIR-Lab/
├── Active-Directory-Attacktive-Directory/
│   └── README.md
├── Windows_Registry-Forensics/
├── docs/
│   ├── commands.md
│   ├── smb_setup.md
│   ├── troubleshooting.md
│   ├── sysmon.md
│   ├── windows-security-events.md
│   └── samba.md
├── images/
├── README.md
└── .gitignore
DisclaimerThis project was created as a controlled laboratory environment for cybersecurity, SOC and DFIR learning.All authentication attempts, network connections and investigative activity described here were performed in an isolated lab environment. No unauthorized systems were targeted.
