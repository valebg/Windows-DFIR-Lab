# SMB / Parrot OS Authentication Test

## Lab objective

Use a Parrot OS workstation to access a Windows SMB share and then investigate the resulting authentication telemetry on the Windows endpoint.

## Example SMB share discovery

```bash
smbclient -L //WINDOWS_IP -U LAB_USER
```

## Connect to the share

```bash
smbclient //WINDOWS_IP/DFIR_Logs -U LAB_USER
```

Then:

```text
ls
```

The lab produced a share containing EVTX files and other test data.

## What Windows records

A failed SMB authentication can produce Security Event ID `4625`.

A successful network authentication can produce Security Event ID `4624` with:

```text
Logon Type: 3
```

The event can provide useful network context such as:

- Workstation name
- Source IP
- Source port
- Authentication package
- Account
- Failure status/substatus

## DFIR lesson

The interesting part is not simply:

> "SMB login failed."

It is:

> "A network authentication request originated from this workstation/IP, targeted this account, used this authentication mechanism, and failed for this specific reason."

That is the difference between reading a log and performing an investigation.
