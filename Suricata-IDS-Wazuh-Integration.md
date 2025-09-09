# Suricata IDS Integration with Wazuh

## Objective
Deploy Suricata IDS with the Emerging Threats ruleset and integrate it with Wazuh to detect malicious network activity and forward alerts to a centralized SIEM.

## Tools & Technologies
- Suricata IDS
- Wazuh Agent & Manager
- Emerging Threats Ruleset
- pfSense for network segmentation
- DVWA (Debian) as victim server
- Kali Linux as attacker

## Setup & Process
- Installed Suricata on Ubuntu server.
- Configured `suricata.yaml` with HOME_NET set to the lab subnet (192.168.3.0/24) and enabled `af-packet` on the monitoring interface.
- Downloaded and enabled Emerging Threats ruleset with `suricata-update`.
- Configured Wazuh agent to forward `/var/log/suricata/eve.json` into Wazuh Manager.
- Validated setup by running simulated attacks and reviewing alerts in the Wazuh dashboard.

## Simulations / Use Cases
- Network probing with Nmap scans from Kali against DVWA.
- SQL injection attempts on DVWA’s web application.
- Detection of suspicious DNS and HTTP traffic.

## Outcomes
- Verified Suricata alert generation with ET rules.
- Successfully integrated Suricata logs into Wazuh SIEM.
- Demonstrated visibility into network-based threats in a lab environment.
- Strengthened detection engineering skills by mapping attacks to Suricata signatures.

## Lessons Learned
- IDS/IPS rules require careful tuning to reduce noise and false positives.
- Integration with a SIEM is essential to centralize analysis and correlate events.
- Combining host logs and network IDS logs increases overall visibility and detection capability.
