# Scenario 2: Brute Force — T1110

## Attacker action (Kali Linux 2026)

```bash
hydra -l administrator -P /usr/share/wordlists/rockyou.txt rdp://192.168.56.20
```

Password-spraying the local Administrator account over RDP using a common wordlist.

## Telemetry generated

Each failed attempt generates a Windows Security **Event ID 4625** (An account failed to log on) on the victim, forwarded by the Wazuh agent. Wazuh's built-in decoder classifies this as SID `60122`.

## Detection

Custom rule `100102` uses Wazuh's `frequency`/`timeframe`/`same_source_ip` correlation to fire a level-10 alert when 8+ failed logons arrive from the same source IP within 120 seconds — a classic brute-force threshold pattern.

## Screenshot checklist

- [ ] `screenshots/attack-simulations/` — Hydra running against the victim
- [ ] `screenshots/alerts-triage/` — Wazuh alert firing in real time alongside the attack

## Analyst notes

This is one of the most common alerts a Tier-1 SOC analyst triages daily. Key questions: is the source IP internal or external, is the target a privileged account, and did any attempt eventually succeed (Event ID 4624 immediately following a burst of 4625s is a major red flag worth escalating).
