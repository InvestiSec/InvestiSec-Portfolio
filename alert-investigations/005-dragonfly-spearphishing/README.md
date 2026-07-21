# Case 005 — Dragonfly Spearphishing Attachment Analysis

## Analyst Summary

I analyzed a Dragonfly-themed spearphishing artifact associated with MITRE ATT&CK T1598.002. The investigation combined email metadata, attachment structure, hash calculation, and embedded-property review to identify delivery infrastructure and a malicious URL.

The message and attachment were assessed as malicious, but the available evidence was limited to email and file analysis. I did not claim endpoint execution or compromise because no endpoint telemetry was provided. This distinction matters: T1598.002 is a reconnaissance technique for phishing to obtain information, whereas T1566.001 describes spearphishing attachments used for initial access.

---

## Investigation Path

| Step | Pivot | Reasoning |
|---|---|---|
| 1 | Email artifact structure | Used `oledump.py` to expose embedded streams and properties not visible in the normal message view. |
| 2 | Sender and transport metadata | Identified the latest IP that contacted the victim and reconstructed message flow. |
| 3 | Endpoint-to-endpoint timing | Calculated the delay between the last two delivery hops to support timeline analysis. |
| 4 | Attachment hash | Calculated MD5 as a lab identifier for correlation and enrichment. |
| 5 | Attachment properties | Extracted the malicious URL associated with the phishing operation. |

---

## Evidence Assessment

| Evidence | Assessment |
|---|---|
| Suspicious attachment | Established the primary phishing artifact. |
| Embedded structure and properties | Revealed information unavailable in the ordinary message view. |
| Sender/transport IP | Supported infrastructure analysis, but required header-path context to interpret correctly. |
| Delivery-hop timing | Helped reconstruct message flow; delay alone was not proof of maliciousness. |
| MD5 hash | Stable file identifier, though SHA-256 would be preferred for modern documentation. |
| Embedded malicious URL | Strongest content-level indicator supporting the malicious verdict. |

---

## Decision

**True Positive — malicious spearphishing attachment. Endpoint compromise not established.**

The email, attachment, and embedded URL support a malicious phishing verdict. Without endpoint, browser, authentication, or network telemetry showing user interaction, the investigation cannot establish successful credential collection or code execution.

---

## Response Actions

1. Quarantine the message and attachment across all mailboxes.
2. Block and hunt for the sender, sending infrastructure, attachment hashes, and embedded URL.
3. Review SPF, DKIM, DMARC, `Received` headers, and reply-path anomalies.
4. Search proxy, DNS, browser, and identity telemetry for user interaction with the URL.
5. Reset credentials and revoke sessions if credential submission is confirmed or strongly suspected.
6. Detonate the attachment in an isolated environment if policy permits and more behavior is needed.
7. Preserve the original message and attachment for repeatable analysis.

---

## MITRE ATT&CK Mapping

| Observed behavior | Technique |
|---|---|
| Attachment used to elicit information during targeting | T1598.002 — Phishing for Information: Spearphishing Attachment |
| Attachment used to gain execution on a victim endpoint | T1566.001 — Phishing: Spearphishing Attachment, only if initial-access intent or execution evidence is established |

---

## Technical Lesson

Email metadata and attachment indicators can prove that a message is malicious without proving that the endpoint was compromised. A defensible phishing investigation keeps delivery, user interaction, credential submission, and endpoint execution as separate questions.
