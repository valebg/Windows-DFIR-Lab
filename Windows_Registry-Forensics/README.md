# Windows Registry Forensics & Artifact Analysis (TryHackMe)

## Overview
This write-up covers a comprehensive digital forensics investigation analyzing core Windows Registry hives (`SAM`, `SOFTWARE`, `SYSTEM`, `NTUSER.DAT`, and `Amcache.hve`). The primary objective was to reconstruct user activity, trace program execution history, and identify external hardware connections using **Registry Explorer** (EZ Tools).

---

## Tools Used
* **Registry Explorer** (Eric Zimmerman's EZ Tools)
* **KAPE** (Kroll Artifact Parser and Extractor)

---

## Key Findings & Investigation Steps

### 1. Account Discovery & Enumeration (`SAM`)
* Parsed the `SAM` hive to map active domain/local users and their associated SIDs/RIDs.
* Examined logon counts, last login timestamps, and password hints for user accounts (`THM-4n6`).

### 2. Execution & File Access History (`NTUSER.DAT` & `Amcache.hve`)
* **Recent Files (`NTUSER.DAT`):** Analyzed `Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` to identify recently accessed documents (e.g., `Changelog.txt`).
* **Binary Execution Paths (`Amcache.hve`):** Located the execution path for installed tools via `InventoryApplicationFile`. 
* **Key Finding:** Identified that the Python 3.8.2 installer was executed from a non-standard network share mapping:
  `Z:\Setups\python-3.8.2.exe`

### 3. Peripheral & USB Tracking (`SYSTEM` & `SOFTWARE`)
* **Device Identification:** Cross-referenced `SOFTWARE\Microsoft\Windows Portable Devices\Devices` and `SYSTEM\ControlSet001\Enum\USBSTOR` to map the Friendly Name (`USB`) to its unique physical serial number.
* **Serial Number:** Extracted the precise serial number (`3C4A92B19A12BDC17C020040`) directly from the `USBSTOR` device path key.
* **Connection Timestamp:** Determined the exact last connection timestamp (`2021-11-24 18:40:06`).

---

## Key Takeaways
* **Raw Registry Precision:** High-resolution artifact inspection within `USBSTOR` keys eliminates ambiguity when distinguishing similar alphanumeric characters in serial numbers.
* **Multi-Hive Correlation:** Correlating `SYSTEM` and `SOFTWARE` hives is essential for binding generic device labels (Friendly Names) to unique physical hardware identifiers.

