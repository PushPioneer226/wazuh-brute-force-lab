# wazuh-brute-force-lab
Home-lab SIEM project demonstrating SSH brute-force detection with Wazuh
# Wazuh SIEM Lab — SSH Brute Force Detection

A home-lab SIEM project demonstrating end-to-end detection engineering with **Wazuh**. This lab simulates a real-world credential attack against a monitored Linux endpoint, detects it in real time, and documents the full investigation as a formal incident report.

---

## 🎯 Project Overview

This project walks through the complete SOC workflow:

1. **Deploy** a Wazuh SIEM stack (Manager + Indexer + Dashboard)
2. **Enroll** a monitored Ubuntu Server endpoint via the Wazuh agent
3. **Simulate** an SSH brute-force attack using Hydra
4. **Detect** the attack via Wazuh's built-in rules and MITRE ATT&CK mapping
5. **Investigate** the alerts and document findings as a formal incident report

The result is a reproducible detection engineering exercise that mirrors the daily responsibilities of a SOC analyst.

---

## 🏗️ Lab Architecture
