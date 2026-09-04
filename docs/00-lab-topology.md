# Lab Topology

## Virtual Machines

| VM | Role | OS | Specs (min) | Network |
|---|---|---|---|---|
| Attacker | Offensive tooling | Kali Linux 2026.3 | 6 vCPU / 4.9GB RAM / 50GB | `192.168.211.128` |
| Victim | Endpoint under monitoring | Windows 10 (22H2) | 4 vCPU / 4GB RAM / 50GB | `192.168.211.129` |
| SOC-Server | Wazuh Manager + ELK | Ubuntu Server 22.04.4 LTS | 4 vCPU / 5.3GB RAM / 62GB | `192.168.211.130` |

All three run on an isolated **host-only / internal network** in VirtualBox or VMware — never bridged to your home LAN. This keeps the brute-force/exploit traffic contained.

## Data Flow

1. **Windows 10 victim** runs the Wazuh agent + Sysmon + osquery agent.
2. Sysmon and Windows Event Log entries are read by the Wazuh agent and forwarded to the **Wazuh Manager**.
3. osquery results are scheduled and shipped either via the Wazuh osquery integration or a Filebeat osquery module.
4. Wazuh Manager applies decoders/rules, generates alerts, and forwards enriched data to **Elasticsearch**.
5. **Kibana** visualizes alerts, and you build dashboards per attack category (auth failures, process creation anomalies, network scans).

## Why this topology

This mirrors a scaled-down version of a real enterprise SOC: an endpoint generating telemetry, a manager doing correlation/detection, and an analytics layer for triage — the exact pipeline described in Wazuh's and Elastic's own reference architectures. It's small enough to run on a single laptop with 16GB RAM, but the components map 1:1 to production tooling.
