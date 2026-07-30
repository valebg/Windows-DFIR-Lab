# Windows DFIR Lab

## Overview

This project demonstrates how to build a small Digital Forensics and Incident Response (DFIR) laboratory for collecting Windows Event Logs from a Windows workstation to a Parrot Security OS analysis workstation using SMB.

The objective is to create a safe environment for Security Operations Center (SOC) training and Windows Event Log analysis.

---

## Lab Environment

| Component | Description |
|-----------|-------------|
| Operating System | Windows 11 |
| Analysis Workstation | Parrot Security OS |
| File Sharing | SMB (Samba) |
| Event Logs | Security, System, Application |
| User Account | soclab |

---

## Lab Architecture

```
+---------------------+
| Windows 11          |
| Event Viewer        |
| DFIR_Logs Share     |
+----------+----------+
           |
           | SMB
           |
+----------v----------+
| Parrot Security OS  |
| smbclient           |
| EVTX Analysis       |
| grep / awk / less   |
+---------------------+
```

---

## Objectives

- Export Windows Event Logs
- Configure SMB file sharing
- Create a dedicated DFIR account
- Access logs remotely from Linux
- Prepare EVTX files for forensic analysis

---

## Technologies

- Windows 11
- Linux
- Parrot Security OS
- Samba (SMB)
- Windows Event Viewer
- PowerShell
- EVTX

---

## Lessons Learned

- Windows Hello PIN cannot be used for SMB authentication.
- Creating a dedicated laboratory account improves security.
- SMB2/SMB3 work correctly even if SMB1 is disabled.
- Windows Event Logs can be collected remotely without modifying the source system.

---

## Next Steps

- EVTX parsing
- Windows Event Log analysis
- Sigma Rules
- Chainsaw
- Hayabusa
- MITRE ATT&CK mapping

