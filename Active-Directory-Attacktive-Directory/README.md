# Active Directory Security Assessment & Exploitation (Attacktive Directory)

## Overview
This laboratory project demonstrates a full-cycle penetration testing and security assessment methodology against a Windows Active Directory environment. The goal of this assessment was to identify misconfigurations, perform privilege escalation, and simulate realistic attack vectors to compromise the Domain Controller.

---

## Technical Stack & Tools Used
* **OS:** Parrot OS (Linux)
* **Enumeration:** `Kerbrute`, `smbclient`
* **Credential Harvesting:** `Impacket` (`GetNPUsers.py`, `secretsdump.py`)
* **Password Cracking:** `Hashcat`
* **Lateral Movement & Access:** `Evil-WinRM`

---

## Attack Lifecycle & Execution Steps

### 1. Domain & User Enumeration
* Performed initial user enumeration against Kerberos (port 88) using `Kerbrute` and a tailored user list.
* Identified valid domain account structures without triggering account lockout policies.

### 2. AS-REP Roasting (Initial Access)
* Identified accounts with Kerberos Pre-Authentication disabled (`DONT_REQUIRE_PREAUTH`).
* Used Impacket's `GetNPUsers.py` to request AS-REP ticket hashes for vulnerable domain users.
* Cracked the Kerberos AS-REP hash offline using `Hashcat` (Mode `18200`) with wordlist mutation to retrieve valid user credentials.

### 3. SMB Enumeration & Secret Discovery
* Authenticated to Server Message Block (SMB) services using `smbclient` with domain user credentials.
* Enumerated non-standard network shares and located a sensitive backup configuration file.
* Decoded encoded secrets (Base64) from the backup share to uncover dedicated Domain Controller service account credentials (`backup`).

### 4. Privilege Escalation via DCSync
* Leveraged the `backup` service account's Directory Replication Service (DRS) privileges.
* Executed a **DCSync attack** via `impacket-secretsdump` to simulate Domain Controller replication.
* Successfully extracted the `NTDS.DIT` database containing NTLM password hashes for all domain users, including the `Administrator` account.

### 5. Post-Exploitation & Pass-the-Hash (PtH)
* Utilized the extracted Domain Administrator NTLM hash to perform a **Pass-the-Hash** attack.
* Established a secure PowerShell remote management session to the Domain Controller using `Evil-WinRM`.
* Verified full system compromise and domain administrative access.

---

## Mitigation & Security Recommendations
1. **Kerberos Pre-Authentication:** Ensure `UF_DONT_REQUIRE_PREAUTH` is disabled for all user accounts across Active Directory.
2. **Access Control on Shares:** Audit SMB share permissions to prevent sensitive backup files and credentials from being readable by domain accounts.
3. **DCSync Rights Auditing:** Restrict `DS-Replication-Get-Changes` and `DS-Replication-Get-Changes-All` permissions to authorized Domain Controllers only.
4. **Strong Password Policies:** Implement strong password complexity and length requirements to resist offline dictionary attacks on Kerberos hashes.
