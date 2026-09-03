# Scenario 1: Network Service Discovery — T1046

## Attacker action (Kali Linux 2026)

```bash
nmap -sS -sV -O -p- 192.168.56.20 -oN nmap_win10_scan.txt
```

A SYN scan with service/version and OS detection against every port on the Windows 10 victim.

## Telemetry generated

Sysmon **Event ID 3** (Network connect) fires repeatedly on the victim as each probed port receives a connection attempt, each logged with source IP, destination port, and protocol.

## Detection

Wazuh rule `100101` (see [`configs/wazuh-rules/local_rules.xml`](../../configs/wazuh-rules/local_rules.xml)) correlates a burst of Event ID 3 logs hitting common service ports (21, 22, 23, 80, 139, 443, 445, 3389) from a single source and raises a level-7 alert.

## Screenshot checklist

- [ ] `screenshots/attack-simulations/` — Nmap scan running in Kali terminal
- [ ] `screenshots/wazuh-dashboard/` — the resulting alert in the Wazuh dashboard

## Analyst notes

In a real environment, a full port sweep like this from an internal host is almost always worth a look — either an authorized vuln-scan or early-stage reconnaissance by an attacker who has already gained a foothold. The follow-up question for triage is always: *is this source IP an approved scanner, and does it correlate with any other suspicious activity in the same time window?*
