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

```mermaid
graph TB
    subgraph HOST["🖥️ Windows Host Machine — VirtualBox Hypervisor"]
        direction TB

        subgraph WAZUH["Wazuh Manager VM<br/>Ubuntu Desktop 24.04<br/>10.0.0.58"]
            WM["Wazuh Manager"]
            WI["Wazuh Indexer"]
            WD["Wazuh Dashboard"]
            WM --> WI
            WI --> WD
        end

        subgraph TARGET["Ubuntu-Target VM<br/>Ubuntu Server 24.04<br/>10.0.0.92"]
            WA["Wazuh Agent"]
            SSH["SSH Service :22"]
        end

        HYDRA["🔴 Hydra Brute Force<br/>(executed from Wazuh VM)"]
    end

    HYDRA -.->|"SSH Brute Force :22"| SSH
    WA -->|"Logs forwarded :1514"| WM

    style HOST fill:#f5f5f5,stroke:#333,stroke-width:2px
    style WAZUH fill:#DAE8FC,stroke:#6C8EBF,stroke-width:2px
    style TARGET fill:#D5E8D4,stroke:#82B366,stroke-width:2px
    style HYDRA fill:#F8CECC,stroke:#B85450,stroke-width:2px
    style WM fill:#ffffff,stroke:#6C8EBF
    style WI fill:#ffffff,stroke:#6C8EBF
    style WD fill:#ffffff,stroke:#6C8EBF
    style WA fill:#ffffff,stroke:#82B366
    style SSH fill:#ffffff,stroke:#82B366
```

**Note on Attacker Placement:** Due to host hardware constraints (12 GB RAM), the brute-force attack was executed from the Wazuh Manager VM itself rather than from a dedicated Kali Linux VM. The detection logic and alert behavior are functionally identical to an external attack. In a fully-resourced lab, a separate Kali instance would originate the attack.

---

## 🧰 Technology Stack

| Component | Technology |
|-----------|------------|
| **SIEM** | Wazuh 4.12 (Manager, Indexer, Dashboard) |
| **Monitored Endpoint** | Ubuntu Server 24.04 |
| **Attacker Tool** | Hydra v9.5 |
| **Hypervisor** | VirtualBox (Bridged Networking) |
| **Framework** | MITRE ATT&CK |

---

## 🔍 Detection Summary

The simulated attack triggered the following Wazuh detection rules:

| Rule ID | Description | Significance |
|---------|-------------|--------------|
| 5710 | Attempt to login using a non-existent user | Reconnaissance / failed attempts |
| 5712 | Multiple authentication failures from same source | Brute-force pattern |
| 5760 | sshd: authentication failed | Individual failed login |
| 5763 | sshd: brute-force attempt | Built-in brute-force detection |
| **5715** | **sshd: authentication success** | **Successful compromise** |

The critical signal is the **authentication success** alert that follows a burst of failures — the canonical indicator of a successful brute-force compromise.

---

## 🎯 MITRE ATT&CK Mapping

| Technique ID | Name | Tactic |
|--------------|------|--------|
| **T1110** | Brute Force | Credential Access |
| T1110.001 | Password Guessing | Credential Access |
| **T1078** | Valid Accounts | Initial Access / Persistence |
| T1021.004 | SSH | Lateral Movement |

---

## 📁 Repository Contents

| File | Description |
|------|-------------|
| [`incident-report.pdf`](./incident-report.pdf) | Full incident report with detection analysis and recommendations |
| [`screenshots/`](./screenshots/) | Evidence: hydra output, Wazuh alerts, MITRE view, agent status |

---

## 🚀 Reproducing the Lab

### Prerequisites

- VirtualBox installed on the host machine
- Ubuntu Desktop 24.04 ISO (for Wazuh Manager)
- Ubuntu Server 24.04 ISO (for monitored endpoint)
- At least 8 GB RAM (12 GB recommended)

### High-Level Steps

**1. Deploy the Wazuh stack** on Ubuntu Desktop:

```bash
sudo curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

**2. Enroll the target** by installing the Wazuh agent on the Ubuntu Server endpoint and pointing it at the manager's IP.

**3. Simulate the attack** using Hydra from the attacker host:

```bash
hydra -l vboxuser -P ~/mini-list.txt ssh://10.0.0.92
```

**4. Investigate** the resulting alerts in the Wazuh dashboard under **Security Events** and **MITRE ATT&CK**.

Full details, screenshots, and analysis are in the [incident report](./incident-report.pdf).

---

## 📚 Key Takeaways

- **Detection works out of the box.** Wazuh's default ruleset correctly identified the brute-force pattern and the subsequent successful login without any custom configuration.
- **Detection is not response.** Wazuh flagged the attack but took no defensive action. Production environments should pair SIEM with active response or SOAR.
- **MITRE ATT&CK mapping adds context.** Automatic technique tagging makes triage faster and more meaningful.

---

## 🔮 Future Enhancements

- [ ] Custom Wazuh rule for "N failures followed by success within X minutes"
- [ ] Wazuh Active Response to auto-block source IPs on brute-force detection
- [ ] Alert forwarding to Slack or email
- [ ] Add a second monitored endpoint (web server) with Apache/Nginx log ingestion
- [ ] Deploy a dedicated Kali attacker VM in a higher-resource environment

---

## 📬 Contact

**Trevor** — Aspiring SOC Analyst | Blue Team | Detection Engineering

- GitHub: [@PushPioneer226](https://github.com/PushPioneer226)

---

*This project is intended for educational purposes only. All attacks were performed against systems I own and operate in an isolated home lab.*

