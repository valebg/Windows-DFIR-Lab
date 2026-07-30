# SMB Lab Setup

## Goal

Collect Windows Event Logs remotely from a Windows workstation using SMB.

## Environment

- Windows 11
- Parrot Security OS
- SMB (Samba)
- Local Network

## Steps

1. Created a shared folder named `DFIR_Logs`.
2. Exported Windows Event Logs:
   - Security.evtx
   - System.evtx
   - Application.evtx
3. Created a dedicated user account `soclab`.
4. Shared the folder with the `soclab` account.
5. Connected from Parrot OS using `smbclient`.
6. Verified remote access to the EVTX files.

## Result

Remote log collection works successfully over the local network.
