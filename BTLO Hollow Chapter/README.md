# Hollow Chapter — Blue Team Labs Online (Hard, 100/100)

> **Disclaimer:** This write-up documents the investigative methodology, artifact correlation workflow, and high-level attack chain observed during the **Hollow Chapter** lab on Blue Team Labs Online (completed **November 16, 2026**, score **100/100**). No flags, challenge answers, extracted payloads, or step-by-step answer keys are included.

## Overview

**Hollow Chapter** is a Hard-level Digital Forensics and Incident Response (DFIR) and malware investigation on Blue Team Labs Online. The scenario follows a multi-stage Windows compromise starting from a deceptive download and progressing through script execution, payload handling, .NET component analysis, persistence mechanisms, and Remote Access Trojan (RAT) / Command-and-Control (C2) activity.

Rather than relying on a single indicator, the investigation focused on correlating independent forensic artifacts across the host to reconstruct a coherent timeline and trace how each stage handed off execution.

## Investigation Flow & Attack Stages

1. **Initial Access & User Execution**
- Examining browser and download activity tied to a lookalike site / purported ebook lure.
- Tracing archive extraction, Windows shortcut (`.lnk`) execution, hidden companion files, and Living-off-the-Land Binaries (LOLBins) launching PowerShell.
2. **Script & Payload Processing (Branch A)**
- Analyzing obfuscated/encrypted scripts and staged data.
- Reconstructing fragmented/hex-encoded pieces, archives, and examining resulting binaries/hashes.
3. **.NET Execution Analysis (Branch B)**
- Inspecting reflection-based assembly loading, deobfuscating .NET components, and analyzing RAT configuration and C2 indicators.
4. **File System & Timeline Correlation**
- Correlating NTFS/MFT metadata, file creation/deletion activity, short-lived log recovery, and decoded timestamps to establish sequence and causality.
5. **Persistence & Remote Access**
- Investigating Windows Scheduled Task registration/modifications alongside persistence and C2 behavior.

## Artifacts & Tools Used

- **Disk / Evidence & NTFS/MFT Analysis:** FTK Imager, MFT Explorer
- **Timeline & Timestamp Decoding:** DCode, filesystem metadata correlation
- **Script & Data Processing:** PowerShell, Python, text editors
- **.NET Analysis & Deobfuscation:** dnSpy, de4dot

## Key Takeaways

- **Context beats isolated IOCs:** Identifying a suspicious file or script is only the start; proving how stages connect requires cross-checking multiple artifact sources.
- **Timeline discipline matters:** Aligning MFT/filesystem timestamps with execution traces and malware artifacts prevents false assumptions about order of operations.
- **Multi-branch visibility:** Complex infections often split into parallel artifact trails (script/payload staging vs. runtime/.NET/C2 behavior) that must be unified into one narrative.

## Evidence & Screenshots

- Completion & score proof (`100/100`, `Hard`, `Nov 16, 2026`): `./screenshots/`
