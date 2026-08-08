# Windows Security Events — Practical SOC Notes

## 4624 — Successful Logon

Use it to establish that an authentication succeeded.

Important investigation fields commonly include:

- Account
- Logon Type
- Logon ID
- Source network information
- Authentication package

Common Logon Types:

- `2` — Interactive
- `3` — Network
- `5` — Service
- `10` — RemoteInteractive / RDP

## 4625 — Failed Logon

Use it to investigate unsuccessful authentication.

Example from the lab:

```text
Logon Type: 3
Workstation: PARROT
Source IP: 192.168.x.x
Authentication Package: NTLM
```

The lab also demonstrated failure substatuses such as:

```text
0xC000006A  -> bad password
0xC0000064  -> bad username
```

These values must be interpreted together with the surrounding event context.

## 4672 — Special Privileges Assigned

This indicates that special privileges were assigned to a new logon.

Do not automatically treat every 4672 as malicious. SYSTEM and other legitimate privileged contexts can generate it.

The SOC question is:

> Which account received the privileges, which Logon ID is associated with it, and what happened afterward?

## Correlation

A useful investigation pattern is:

```text
4624 / 4625
      |
      v
authentication context
      |
      +----> 4672
      |
      v
Sysmon Event 1
      |
      v
process / parent / command line
```

The exact chain depends on what actually occurred on the endpoint; do not claim an event occurred unless the logs support it.
