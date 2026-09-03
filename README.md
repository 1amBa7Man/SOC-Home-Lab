# 🛡️ SOC Home Lab — Detection Engineering & Threat Simulation Environment

[![Made with Wazuh](https://img.shields.io/badge/SIEM-Wazuh-3aa5d6)](https://wazuh.com/)
[![ELK Stack](https://img.shields.io/badge/Stack-ELK%2FKibana-005571)](https://www.elastic.co/)
[![Sysmon](https://img.shields.io/badge/Telemetry-Sysmon-blue)](https://learn.microsoft.com/sysinternals/downloads/sysmon)
[![osquery](https://img.shields.io/badge/EDR-osquery-purple)](https://osquery.io/)
[![Attacker OS](https://img.shields.io/badge/Attacker-Kali%20Linux%202026-557C94)](https://www.kali.org/)
[![Target OS](https://img.shields.io/badge/Target-Windows%2010-0078D6)](https://www.microsoft.com/windows)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

> A fully self-built Security Operations Center lab that generates real attacker telemetry, ingests it into a SIEM, and turns it into detections, triage workflows, and written incident reports — the same loop an entry-level SOC Analyst runs in production.

**Author:** Paramananda Gaigaria — B.Tech CSE , aspiring SOC Analyst
🔗 [LinkedIn](#) · [TryHackMe](#) · [Resume](#)

---

## 🎯 Why this project exists

Most "home lab" repos stop at *"I installed a SIEM."* This one is built to answer the question every SOC interviewer actually asks: **"Walk me through an alert you triaged, end to end."**

Every attack in this lab follows the same four-stage loop:

`Simulate an attack (Kali) → Generate telemetry (Sysmon/osquery) → Detect it (Wazuh/ELK rule) → Write it up (triage note mapped to MITRE ATT&CK)`

---

## 🏗️ Architecture

```
┌─────────────────────┐        Attacks (T1110, T1059, T1003...)      ┌──────────────────────────┐
│   Kali Linux 2026    │ ───────────────────────────────────────────▶ │   Windows 10 Victim VM   │
│   (Attacker)         │        Nmap · Hydra · Metasploit · CrackMap  │   Sysmon + osquery agent │
└─────────────────────┘                                               └────────────┬─────────────┘
                                                                                     │ forwarded logs
                                                                                     ▼
                                                                       ┌──────────────────────────┐
                                                                       │      Wazuh Manager        │
                                                                       │  (decoders + custom rules) │
                                                                       └────────────┬─────────────┘
                                                                                     │
                                                                                     ▼
                                                                       ┌──────────────────────────┐
                                                                       │   ELK Stack (Kibana)      │
                                                                       │ dashboards · alerts · SOAR│
                                                                       └──────────────────────────┘
```

Full diagram + IP/network layout: [`docs/architecture.md`](docs/architecture.md)

---

## 🧰 Lab Components

| Layer | Tool | Purpose |
|---|---|---|
| Attacker | Kali Linux 2026 | Offensive tooling — Nmap, Hydra, Metasploit, CrackMapExec, Mimikatz delivery |
| Victim (Endpoint) | Windows 10 | Domain-joined-style target with real user activity baseline |
| Host Telemetry | Sysmon | Process creation, network conns, registry, image loads (config in [`configs/sysmon`](configs/sysmon)) |
| Endpoint Visibility | osquery | Scheduled + live queries for persistence, autoruns, running processes |
| SIEM / HIDS | Wazuh | Log collection, FIM, custom decoders/rules, active response |
| Log Analytics | ELK (Elasticsearch, Logstash, Kibana) | Dashboards, alert visualization, long-term log storage |

---

## 🗂️ Repository Structure

```
soc-home-lab/
├── docs/                        # Architecture, setup guides, lessons learned
├── configs/
│   ├── sysmon/                  # sysmonconfig.xml (event filtering ruleset)
│   ├── wazuh-rules/             # Custom Wazuh decoders & detection rules
│   └── osquery/                 # osquery.conf query packs
├── attack-simulations/
│   └── mitre-mapped/            # Each attack scenario mapped to MITRE ATT&CK TTPs
├── reports/
│   └── sample-incident-report/  # Analyst-style write-up template + filled example
└── screenshots/                 # Evidence — dashboards, alerts, terminal captures
    ├── wazuh-dashboard/
    ├── elk-kibana/
    ├── sysmon-events/
    ├── osquery/
    ├── attack-simulations/
    └── alerts-triage/
```

---

## 🔴 Attack Scenarios Simulated

| # | Scenario | MITRE ATT&CK | Detection Source |
|---|---|---|---|
| 1 | Nmap network/service enumeration against victim | T1046 Network Service Discovery | Wazuh (Sysmon Event ID 3) |
| 2 | Hydra brute-force against RDP/SMB | T1110 Brute Force | Wazuh (Windows Security 4625 correlation rule) |
| 3 | Malicious macro / LOLBins execution (`powershell.exe -enc`) | T1059.001 PowerShell | Sysmon Event ID 1 → Wazuh custom rule |
| 4 | Mimikatz-style credential dumping | T1003 OS Credential Dumping | Sysmon Event ID 10 (lsass.exe access) |
| 5 | Scheduled task persistence | T1053.005 Scheduled Task | osquery `scheduled_tasks` table + Wazuh FIM |
| 6 | C2 beacon simulation (DNS/HTTP) | T1071 Application Layer Protocol | Sysmon Event ID 22 (DNS query) → Wazuh alert |

Each scenario has a full writeup: attacker commands, raw log excerpt, the exact Wazuh/Sigma rule that fired, and a screenshot of the Kibana alert. See [`attack-simulations/mitre-mapped/`](attack-simulations/mitre-mapped/).

---

## 📸 Screenshots

All evidence lives in [`/screenshots`](screenshots/), organized by lab component. **Add your own captures here** — see [`screenshots/README.md`](screenshots/README.md) for the exact list of shots that make this repo interview-ready (Wazuh dashboard, Kibana alert view, raw Sysmon event, osquery result, attacker terminal, and a triaged alert).

---

## 🚀 Getting Started

Step-by-step build guides (VM setup → Wazuh manager/agent → Sysmon deployment → osquery → Kibana dashboards) are in [`docs/`](docs/):

1. [`docs/00-lab-topology.md`](docs/00-lab-topology.md) — VM specs & network layout
2. [`docs/01-setup-wazuh.md`](docs/01-setup-wazuh.md) — Wazuh manager + agent install
3. [`docs/02-setup-elk.md`](docs/02-setup-elk.md) — ELK stack + Kibana dashboards
4. [`docs/03-setup-sysmon.md`](docs/03-setup-sysmon.md) — Sysmon deployment & config
5. [`docs/04-setup-osquery.md`](docs/04-setup-osquery.md) — osquery agent + query packs
6. [`docs/05-detection-engineering.md`](docs/05-detection-engineering.md) — Writing custom Wazuh rules

---

## 📄 Sample Incident Report

A full analyst-style report (executive summary, timeline, IOCs, MITRE mapping, remediation) is included as a template you can point to in interviews: [`reports/sample-incident-report/`](reports/sample-incident-report/).

---

## 🗺️ Roadmap

- [ ] Add Suricata/Snort for network-layer detection
- [ ] Add Sigma → Wazuh rule conversion pipeline
- [ ] Add a SOAR playbook (Shuffle/TheHive) for auto-triage
- [ ] Record a 5-minute walkthrough video for the README

---

## 📜 License

MIT — see [LICENSE](LICENSE). Use this as a template for your own SOC lab.
