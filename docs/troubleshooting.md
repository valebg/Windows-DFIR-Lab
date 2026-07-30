# Troubleshooting

## NT_STATUS_LOGON_FAILURE

**Cause**

Incorrect username or password.

**Solution**

Created a dedicated local account (`soclab`) for SMB authentication.

---

## Windows Hello PIN

Windows Hello PIN cannot be used for SMB authentication.

Use the account password instead.

---

## SMB1 Warning

The warning about SMB1 does not affect SMB2/SMB3 connectivity.

---

## Network Discovery

Enabled:

- Network Discovery
- File and Printer Sharing

---

## Verification

Successfully connected from Parrot OS and listed:

- Security.evtx
- System.evtx
- Application.evtx
