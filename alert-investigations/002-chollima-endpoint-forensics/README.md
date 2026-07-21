# Case 002 — Famous Chollima Fake-Recruitment Compromise

## Analyst Summary

I investigated a training scenario in which a finance employee was approached through a fake recruitment process and instructed to run a PowerShell command presented as a camera-driver installation step. The command downloaded a disguised archive and script payload into the user's temporary directory.

The case escalated to confirmed compromise after file-system, script, registry, and MFT artifacts showed payload staging and persistence through the current user's `Run` key. The decisive evidence was not the recruitment lure by itself, but the correlation between user-driven PowerShell execution, the extracted payloads, the startup-registration logic, and the resulting registry entry.

---

## Investigation Path

| Step | Pivot | Reasoning |
|---|---|---|
| 1 | Scenario and PowerShell command | Established how the attacker persuaded the user to initiate execution. |
| 2 | Temporary-directory artifacts | Located the downloaded archive and extracted VBS/Python components. |
| 3 | Static script review | Identified `register_startup` logic in `nvidia.py`, indicating persistence intent. |
| 4 | User registry hives | Loaded `NTUSER.DAT` and its transaction log in Registry Explorer. |
| 5 | `Run` key | Confirmed an autostart entry under `Software\Microsoft\Windows\CurrentVersion\Run`. |
| 6 | `$MFT` timeline | Correlated file activity with the payload-staging and persistence sequence. |

---

## Evidence Assessment

| Evidence | Assessment |
|---|---|
| Attacker-provided PowerShell command | Established user-driven initial execution without requiring a software exploit. |
| `%TEMP%\nvidiaRelease.zip` | Vendor-themed archive name was consistent with masquerading. |
| `%TEMP%\nvidiaRelease\update.vbs` | Script component associated with the staged payload. |
| `nvidia.py` | Static review exposed the `register_startup` persistence-related logic. |
| User `Run` key | Confirmed persistence at logon rather than merely showing intent in source code. |
| `$MFT` records | Supported the ordering and timing of download, extraction, and persistence artifacts. |

---

## Decision

**True Positive — confirmed endpoint compromise with user-level persistence.**

The evidence supports a coherent chain from social engineering to PowerShell execution, payload staging, and Registry Run-key persistence. The available artifacts do not by themselves establish credential theft, lateral movement, or data exfiltration.

---

## Indicators and Artifacts

> These are lab-derived indicators. They are defanged for safe documentation and should be validated before operational use.

| Type | Value |
|---|---|
| Download URL | `hxxp://3[.]101[.]53[.]248:8080/vcam-installer[.]exe` |
| Archive | `%TEMP%\nvidiaRelease.zip` |
| Script | `%TEMP%\nvidiaRelease\update.vbs` |
| Python component | `nvidia.py` |
| Persistence logic | `register_startup` |
| Registry location | `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` |

---

## Response Actions

1. Isolate the affected endpoint and preserve volatile and disk evidence.
2. Block and hunt for the observed infrastructure across DNS, proxy, firewall, and EDR telemetry.
3. Collect PowerShell, Sysmon, Defender, and Security event logs.
4. Remove the unauthorized Run-key value and payloads only after evidence preservation.
5. Search other endpoints for the archive, script names, registry value, and related command lines.
6. Reset exposed credentials if browser, token, or credential-access evidence is found.
7. Rebuild the endpoint if its integrity cannot be established confidently.

---

## MITRE ATT&CK Mapping

| Observed behavior | Technique |
|---|---|
| PowerShell execution | T1059.001 — Command and Scripting Interpreter: PowerShell |
| VBS payload | T1059.005 — Command and Scripting Interpreter: Visual Basic |
| NVIDIA-like naming | T1036.005 — Masquerading: Match Legitimate Resource Name or Location |
| User Run-key persistence | T1547.001 — Registry Run Keys / Startup Folder |

---

## Technical Lesson

The strongest conclusion came from artifact correlation. The command explained execution, the temporary directory showed staging, script review showed persistence intent, the registry hive confirmed persistence, and the MFT supported the timeline. Any one artifact was incomplete; together they established the compromise chain.
