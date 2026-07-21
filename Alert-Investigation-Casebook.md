# Alert Investigation Casebook

This casebook collects concise alert investigations written from an analyst perspective. Each case focuses on how the alert was triaged, which pivots mattered, what evidence changed the assessment, and what response actions were justified.

The goal is to show investigation judgment, not to reproduce lab answers or publish long reports.

---

## Case Index

| Case | Focus | Status | Link |
|---|---|---|---|
| 001 | Remote Code Execution against Splunk Enterprise | Published | [Open case](./alert-investigations/001-splunk-rce/README.md) |
| 002 | Famous Chollima fake-recruitment compromise | Published | [Open case](./alert-investigations/002-chollima-endpoint-forensics/README.md) |
| 003 | RedLine memory forensics | Published | [Open case](./alert-investigations/003-redline-memory-forensics/README.md) |
| 004 | RevengeHotels multi-stage endpoint compromise | Published | [Open case](./alert-investigations/004-revengehotels-apt/README.md) |
| 005 | Dragonfly spearphishing attachment analysis | Published | [Open case](./alert-investigations/005-dragonfly-spearphishing/README.md) |

---

## Writing Standard

Each case should stay compact and defensible:

- short analyst summary
- clear investigation path
- evidence assessment
- decision and impact
- response actions
- technical lesson learned

Indicators are defanged for safe publication. Hostnames, usernames, and paths are anonymized or retained only when they are necessary lab evidence and do not expose a real person or environment.
