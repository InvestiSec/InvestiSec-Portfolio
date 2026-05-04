# Case 001 — Remote Code Execution Against Splunk Enterprise

## Analyst Summary

I investigated a suspected remote code execution alert against a Splunk Enterprise host. The case escalated from suspicious web activity to confirmed compromise after correlated proxy, firewall, authentication, process, and terminal telemetry showed authenticated access, shell execution, and local account creation.

The strongest finding was not the source reputation or the alert title. The decisive evidence was the sequence of post-exploitation behavior on the host: shell activity, discovery commands, interaction with Splunk script paths, account creation, password assignment, and failed-login counter reset.

---

## Investigation Path

| Step | Pivot | Reasoning |
|---|---|---|
| 1 | Alert context | Reviewed the alert objective and affected service to understand whether the activity targeted application functionality or the operating system directly. |
| 2 | Source enrichment | Used external reputation and ownership only for prioritization, not as proof of compromise. |
| 3 | Proxy and firewall telemetry | Confirmed the suspicious source interacted with the Splunk web endpoint and that the traffic was not limited to scanning noise. |
| 4 | Authentication evidence | Verified that the activity included successful access with valid credentials rather than only failed exploitation attempts. |
| 5 | Endpoint and terminal telemetry | Confirmed host-level shell activity and attacker-executed commands. |
| 6 | Account activity | Identified local user creation, password assignment, and failed-login counter reset as persistence/account manipulation evidence. |

---

## Evidence Assessment

| Evidence Area | Assessment |
|---|---|
| Source reputation | Suspicious and useful for enrichment, but not sufficient to prove impact. |
| Web activity | Suspicious upload behavior against a Splunk web endpoint indicated likely exploitation attempt. |
| Authentication | Successful access showed the activity was not just unauthenticated probing. |
| Shell execution | Terminal/process telemetry confirmed command execution on the host. |
| Discovery commands | Commands such as user and group discovery indicated post-access validation. |
| Splunk script path interaction | Activity under Splunk script-related paths strengthened the link between web exploitation and host execution. |
| Local account creation | Creation of a new local account indicated persistence preparation. |
| Password assignment | Setting a password for the new account made the persistence mechanism usable. |
| Failed-login counter reset | Resetting authentication counters supported account manipulation behavior. |

---

## Decision

**True Positive — confirmed host compromise with persistence activity.**

The available telemetry supports successful compromise of the Splunk host. I would not classify this as a simple exploitation attempt because host-level activity and account manipulation were observed after the web-layer event.

No confirmed lateral movement or data exfiltration was identified from the available evidence. That limitation should be stated clearly rather than assumed.

---

## Response Actions

1. Isolate the affected Splunk host from the network.
2. Preserve logs and forensic evidence before cleanup.
3. Disable the unauthorized local account and review all recent account changes.
4. Reset affected application and operating-system credentials.
5. Review shell history, process execution, authentication logs, and Splunk audit logs.
6. Remove malicious artifacts only after evidence preservation.
7. Validate Splunk patch level, installed apps, exposed services, and access restrictions.
8. Hunt for similar behavior across other assets.
9. Rebuild the host if integrity cannot be trusted after forensic review.

---

## MITRE ATT&CK Mapping

| Behavior | Technique |
|---|---|
| Exploitation of exposed application functionality | Exploit Public-Facing Application |
| Shell command execution | Command and Scripting Interpreter: Unix Shell |
| User and privilege discovery | System Owner/User Discovery / Permission Groups Discovery |
| Local account creation | Create Account: Local Account |
| Password assignment and authentication-counter reset | Account Manipulation |

---

## Technical Lesson

The important lesson from this case is evidence hierarchy. Reputation and alert metadata helped prioritize the investigation, but the final decision came from correlated telemetry across web traffic, authentication, process execution, terminal history, and account activity.

A strong investigation should show exactly where the assessment changed from suspicious activity to confirmed compromise.
