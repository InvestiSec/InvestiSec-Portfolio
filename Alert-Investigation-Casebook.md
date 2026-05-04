# Alert Investigation Casebook

This page documents my alert-analysis workflow, reasoning process, and investigation notes from hands-on SOC scenarios. The goal is not to publish simple answers, but to show how I think through alerts: validating telemetry, separating reputation from evidence, identifying attacker behavior, mapping activity to MITRE ATT&CK, and deciding whether an event is a false positive, attempted attack, or confirmed compromise.

---

## Investigation Principles

When I investigate an alert, I try to separate five layers:

1. **Context** - IP reputation, threat-intelligence enrichment, known abuse history, affected asset, exposed service.
2. **Access Evidence** - authentication status, HTTP response codes, allowed/blocked actions, successful sessions.
3. **Endpoint Evidence** - process execution, parent/child process relationships, command lines, file paths, users, persistence actions.
4. **Impact Evidence** - what the attacker actually achieved: discovery, execution, privilege use, account creation, data access, or lateral movement.
5. **Response Decision** - close, monitor, escalate, isolate, contain, or rebuild.

The most important rule is simple: **reputation makes an alert suspicious; telemetry proves what happened.**

---

## Case Index

| Case | Category | Status | Summary |
|---|---|---|---|
| [Remote Code Execution Against Splunk Enterprise](#remote-code-execution-against-splunk-enterprise) | Unauthorized Access / RCE / Persistence | True Positive | Malicious XSLT upload against Splunk led to shell access, local user creation, password setup, and account manipulation. |

---

## Remote Code Execution Against Splunk Enterprise

### Executive Summary

A high-severity alert identified suspected Remote Code Execution activity against a Splunk Enterprise host. Investigation confirmed that the activity was successful. The attacker abused Splunk upload/indexing preview functionality to upload a malicious XSLT file, gained shell access, executed discovery commands, interacted with Splunk script paths, created a new local user, set a password for that user, and reset failed-login counters.

This should be treated as a confirmed host compromise with persistence activity.

---

### Key Alert Details

| Field | Value |
|---|---|
| Affected Host | Splunk Enterprise |
| Destination IP | 172.16.20.13 |
| Source IP | 180.101.88.240 |
| Request Method | POST |
| Service / Port | Splunk web service on port 8000 |
| Suspicious File | `shell.xsl` |
| Triggered File Path | `/opt/splunk/var/run/splunk/dispatch/.../shell.xsl` |
| Device Action | Allowed |
| Initial Classification | Unauthorized Access / Remote Code Execution |

---

### Investigation Reasoning

The source IP had a poor reputation and was associated with brute-force and SSH-related abuse. This was useful for prioritization, but it was not enough to prove compromise. The investigation needed internal telemetry: proxy logs, firewall logs, process execution, terminal history, file paths, and network activity.

The first important indicator was a POST request to the Splunk upload/indexing preview endpoint. The request included the parameter `input.path=shell.xsl`, which suggested abuse of Splunk's file upload and XSLT processing behavior.

At this stage, the alert was suspicious, but not yet fully proven as a compromise. The investigation became much stronger after endpoint and terminal evidence showed command execution on the Splunk host.

---

### Evidence Timeline

| Time | Evidence | Interpretation |
|---|---|---|
| 12:23 | POST request to Splunk upload/indexing preview endpoint | Initial abuse of Splunk upload/XSLT functionality |
| 12:23 | `input.path=shell.xsl` | Malicious XSLT file used in the attack chain |
| 12:23-12:24 | Shell activity observed | Successful command execution on host |
| 12:23 | `cd /opt/splunk/bin/scripts/` | Attacker moved into Splunk script path |
| 12:24 | `cat shell.sh` | Attacker inspected a shell script connected to the activity |
| 12:24 | `whoami`, `id`, `groups` | Account and privilege discovery |
| 12:24 | `useradd -m analyst` | New local user created |
| 12:24 | `passwd analyst` | Password set for the new account |
| 12:24 | `pam_tally2 --user analyst --reset --quiet` | Failed-login counter reset for the account |

---

### Commands of Interest

```bash
cd /opt/splunk/bin/scripts/
cat shell.sh
whoami
id
groups
useradd -m analyst
passwd analyst
pam_tally2 --user analyst --reset --quiet
```

---

### Why This Is a True Positive

This was not just a suspicious request or a blocked exploit attempt. The strongest evidence is the post-exploitation behavior:

- The attacker obtained shell access.
- Discovery commands were executed.
- Splunk script paths were accessed.
- A new local user named `analyst` was created.
- A password was assigned to the new user.
- Failed-login counters were reset for that user.

Together, these actions prove successful compromise and persistence preparation.

---

### MITRE ATT&CK Mapping

| Technique | ID | Evidence |
|---|---|---|
| Exploit Public-Facing Application | T1190 | Abuse of Splunk web/upload functionality leading to RCE |
| Command and Scripting Interpreter: Unix Shell | T1059.004 | `bash`, terminal commands, shell activity |
| System Owner/User Discovery | T1033 | `whoami`, `id` |
| Permission Groups Discovery | T1069 | `groups` |
| Create Account: Local Account | T1136.001 | `useradd -m analyst` |
| Account Manipulation | T1098 | `passwd analyst`, `pam_tally2 --user analyst --reset --quiet` |

---

### Analyst Assessment

This is a confirmed compromise. The attacker moved from malicious application-layer activity into host-level command execution and persistence. The account creation and password assignment indicate intent to maintain access. The PAM counter reset suggests account manipulation to keep the new account usable or reduce authentication friction.

---

### Response Recommendations

1. Isolate the Splunk Enterprise host from the network.
2. Preserve evidence before cleanup.
3. Disable or remove the unauthorized `analyst` account.
4. Reset credentials for affected Splunk and operating-system accounts.
5. Review Splunk authentication logs, Linux authentication logs, shell history, and process telemetry.
6. Identify and preserve suspicious files such as `shell.xsl` and `shell.sh`.
7. Block the source IP at perimeter controls.
8. Review Splunk version, installed apps, exposed services, and upload/indexing preview functionality.
9. Hunt for similar behavior across other systems.
10. Rebuild the host if forensic review cannot guarantee integrity.

---

### Final Analyst Note

Investigation confirms successful compromise of the Splunk Enterprise host `172.16.20.13`. The suspicious source `180.101.88.240` abused Splunk upload/indexing preview functionality and uploaded a malicious XSLT file, leading to shell access. Endpoint and terminal telemetry show discovery commands, interaction with `/opt/splunk/bin/scripts/`, creation of local user `analyst`, password assignment, and reset of PAM failed-login counters. This indicates unauthorized access, persistence, and account manipulation. Immediate containment, credential reset, forensic review, and host isolation are recommended.

---

## Reusable Alert Write-Up Template

Use this structure for future cases:

```markdown
## Case Title

### Executive Summary
Briefly explain what happened and why it matters.

### Key Alert Details
Asset, source, destination, service, action, alert category.

### Investigation Reasoning
Explain the logic step by step. Separate assumptions from evidence.

### Evidence Timeline
List important events in chronological order.

### Why This Is True Positive / False Positive / Needs Escalation
State the decision and justify it with evidence.

### MITRE ATT&CK Mapping
Map only techniques supported by telemetry.

### Response Recommendations
Containment, eradication, recovery, and hardening.

### Final Analyst Note
A short SOC-style note suitable for escalation or case closure.
```
