# Sysmon — Practical Notes

## Purpose

Microsoft Sysmon provides additional endpoint telemetry in a dedicated Windows Event Log:

`Microsoft-Windows-Sysmon/Operational`

It complements, rather than replaces, native Windows logs such as Security, System, and Application.

## Verification

```powershell
Get-Service Sysmon64

Get-WinEvent -ListLog * |
    Where-Object LogName -like "*Sysmon*"
```

## Read recent Sysmon events

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10 |
    Format-List TimeCreated, Id, Message
```

## Process creation — Event ID 1

Useful fields include:

- `Image`
- `CommandLine`
- `User`
- `LogonId`
- `TerminalSessionId`
- `IntegrityLevel`
- `ProcessId`
- `ProcessGuid`
- `ParentProcessId`
- `ParentImage`
- `ParentCommandLine`
- `ParentUser`
- `SHA256`

Example investigation question:

> Why was this process created, who created it, and what created the parent process?

## Search for PowerShell

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" |
Where-Object { $_.Id -eq 1 -and $_.Message -match "powershell.exe" } |
Select-Object TimeCreated, Message
```

## Search for CMD

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" |
Where-Object { $_.Id -eq 1 -and $_.Message -match "cmd.exe" } |
Select-Object TimeCreated, Message
```

## Important distinction

Sysmon does **not** simply reread Security.evtx and make it more detailed. It generates additional telemetry and writes it to its own event channel.

## Storage

A circular Sysmon event log has a configured maximum size. In this lab it was observed as approximately 64 MiB. Circular logging means new events eventually overwrite older events when the configured limit is reached.

This controls disk usage, but Sysmon still creates some CPU, memory, disk I/O, and—if events are forwarded—network overhead.
