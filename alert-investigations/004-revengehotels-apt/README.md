# Case 004 — RevengeHotels Multi-Stage Endpoint Compromise

## Analyst Summary

I reconstructed a multi-stage training intrusion that began with a malicious email attachment and progressed through browser activity, JavaScript and PowerShell execution, payload reconstruction, security-control modification, RunOnce persistence, command-and-control communication, and data collection.

The decisive evidence was the continuity across independent artifacts: the user opened the attachment, scripts staged obfuscated components under a public directory, a masqueraded executable modified defenses, persistence referenced the copied payload, the host contacted external infrastructure, and collected data was placed in an archive.

---

## Investigation Path

| Step | Pivot | Reasoning |
|---|---|---|
| 1 | Email attachment and Downloads | Established the suspected initial-access artifact. |
| 2 | Chrome history | Connected user browsing with the attack's delivery path. |
| 3 | JavaScript and PowerShell artifacts | Reconstructed the script-based execution chain. |
| 4 | `C:\Users\Public\Scripts` | Located the obfuscated and reconstructed payload components. |
| 5 | Security-related registry keys | Confirmed attempts to disable or weaken endpoint defenses. |
| 6 | RunOnce and VBS artifacts | Confirmed persistence after the initial execution. |
| 7 | External IP and collection files | Identified command-and-control activity and collection staging. |

---

## Evidence Assessment

| Evidence | Assessment |
|---|---|
| Malicious attachment | Supported the initial-access starting point. |
| `venumentrada.txt` and `runpe.txt` | Obfuscated/staged components used to reconstruct an executable payload. |
| `swchost.exe` | Filename closely imitated `svchost.exe`, supporting masquerading. |
| Security registry changes | Confirmed defense impairment rather than a merely attempted action. |
| `RunOnce\svchostAS` and VBS script | Established a persistence mechanism tied to the copied payload. |
| External IP | Linked endpoint activity to remote infrastructure. |
| `Flfs6heTV2lb.exe` and `data.zip` | Supported collection and staging; exfiltration was not established. |
| `RtlSetProcessIsCritical` | Indicated an attempt to make process termination disruptive or difficult. |

---

## Decision

**True Positive — confirmed multi-stage endpoint compromise with persistence, defense evasion, and collection staging.**

The evidence supports successful execution and sustained malicious activity. The creation of `data.zip` shows staging, but the available artifacts do not prove that the archive left the endpoint.

---

## Indicators and Artifacts

> These are lab-derived indicators. They are defanged for safe documentation and should be validated before operational use.

| Type | Value |
|---|---|
| Staging directory | `C:\Users\Public\Scripts` |
| Components | `venumentrada.txt`, `runpe.txt` |
| Masqueraded executable | `swchost.exe` |
| External IP | `3[.]122[.]239[.]15` |
| Persistence payload | `C:\Users\Administrator\AppData\Roaming\host\swchost.exe` |
| Registry value | `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce\svchostAS` |
| VBS component | `KOoNLZeCGlnQ.vbs` |
| Collection component | `Flfs6heTV2lb.exe` |
| Staged archive | `data.zip` |

---

## Response Actions

1. Isolate the endpoint and preserve disk, memory, email, browser, and event-log evidence.
2. Block and hunt for the external IP and all identified filenames, paths, hashes, and registry values.
3. Restore altered security settings and verify endpoint protection health.
4. Remove persistence and malicious components only after evidence preservation.
5. Review PowerShell script-block, Sysmon, Defender, Security, and Task Scheduler telemetry.
6. Determine what entered `data.zip` and search network controls for evidence of transfer.
7. Reset affected credentials and investigate access from the administrator workstation.
8. Rebuild the system if integrity cannot be re-established confidently.

---

## MITRE ATT&CK Mapping

| Observed behavior | Technique |
|---|---|
| Malicious email attachment | T1566.001 — Phishing: Spearphishing Attachment |
| JavaScript/JScript execution | T1059.007 — Command and Scripting Interpreter: JavaScript/JScript |
| PowerShell execution | T1059.001 — Command and Scripting Interpreter: PowerShell |
| `swchost.exe` naming | T1036.005 — Masquerading: Match Legitimate Resource Name or Location |
| Security-tool modification | T1562.001 — Impair Defenses: Disable or Modify Tools |
| RunOnce persistence | T1547.001 — Registry Run Keys / Startup Folder |
| Data archived as `data.zip` | T1560.001 — Archive Collected Data: Archive via Utility, if the archive operation is confirmed |

---

## Technical Lesson

The archive was evidence of staging, not automatic proof of exfiltration. Separating what the artifacts prove from what they merely suggest makes the final assessment stronger and prevents an investigation from overstating impact.
