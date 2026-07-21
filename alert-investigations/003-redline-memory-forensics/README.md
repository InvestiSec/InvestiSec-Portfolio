# Case 003 — RedLine Memory Forensics Investigation

## Analyst Summary

I analyzed a compromised Windows memory image with Volatility 3 and string searches. Process-tree analysis identified `oneetx.exe` as the suspicious process and `rundll32.exe` as its child. Network artifacts linked the activity to an external IP and URL, while memory strings revealed the executable's location under the user's temporary directory.

The case was assessed as a confirmed malicious execution based on the combined process, memory-permission, network, URL, and file-path evidence. A `PAGE_EXECUTE_READWRITE` region increased suspicion, but I did not treat that permission alone as proof of injection.

---

## Investigation Path

| Step | Pivot | Reasoning |
|---|---|---|
| 1 | `windows.pstree` | Identified abnormal processes and parent-child relationships. |
| 2 | Process memory | Reviewed suspicious memory protection and execution context. |
| 3 | `windows.netscan` | Connected runtime process activity with external infrastructure. |
| 4 | IP string search | Recovered a full HTTP URL from the memory image. |
| 5 | Executable-name search | Recovered the full on-disk path of the suspicious binary. |
| 6 | VPN process review | Identified `outline.exe` as the process associated with VPN connectivity. |

---

## Evidence Assessment

| Evidence | Assessment |
|---|---|
| `oneetx.exe` | Primary suspicious process identified in the process tree. |
| Child `rundll32.exe` | Legitimate Windows binary in a suspicious relationship; requires command-line and loaded-module validation. |
| `PAGE_EXECUTE_READWRITE` | Risky memory permission consistent with runtime payload activity, but not proof by itself. |
| External connection | Linked the process environment to attacker-controlled or lab-designated infrastructure. |
| Full URL in memory | Strengthened the network finding and provided a more precise hunt indicator. |
| `%LOCALAPPDATA%\Temp` executable | User-writable staging path increased confidence in malicious execution. |

---

## Decision

**True Positive — confirmed suspicious executable activity with external communication.**

The available memory evidence supports malicious execution and network communication. It does not, by itself, prove the exact malware family, successful credential theft, persistence, or data exfiltration. Those conclusions would require additional host and network evidence.

---

## Indicators and Artifacts

> These are lab-derived indicators. They are defanged for safe documentation and should be validated before operational use.

| Type | Value |
|---|---|
| Process | `oneetx.exe` |
| Child process | `rundll32.exe` |
| VPN-related process | `outline.exe` |
| Memory protection | `PAGE_EXECUTE_READWRITE` |
| IP address | `77[.]91[.]124[.]20` |
| URL | `hxxp://77[.]91[.]124[.]20/store/games/index[.]php` |
| Executable path | `C:\Users\Tammam\AppData\Local\Temp\c3912af058\oneetx.exe` |

---

## Response Actions

1. Isolate the endpoint represented by the memory image.
2. Preserve the memory image and acquire disk and relevant event logs.
3. Block and hunt for the observed IP, URL, executable name, hash, and path pattern.
4. Inspect `rundll32.exe` command-line arguments, loaded modules, and process ancestry.
5. Dump and analyze suspicious process memory and executable content in an isolated environment.
6. Correlate with Sysmon, Security, Defender, DNS, proxy, and firewall telemetry.
7. Reset credentials if follow-up analysis identifies credential-access behavior.

---

## MITRE ATT&CK Mapping

| Observed behavior | Technique |
|---|---|
| Execution from a user temporary directory | T1204 — User Execution, if initiated by the user; initial vector not established in the memory evidence |
| `rundll32.exe` child process | T1218.011 — Rundll32, pending validation that it executed malicious content |
| HTTP communication | T1071.001 — Application Layer Protocol: Web Protocols |

---

## Technical Lesson

Memory permissions, process names, and single network indicators are not decisive in isolation. The defensible assessment came from correlating the process tree, risky memory region, external connection, recovered URL, and executable path while keeping unsupported conclusions explicitly out of scope.
