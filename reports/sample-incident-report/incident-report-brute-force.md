# Incident Report — RDP Brute Force Against WIN10-VICTIM

| Field | Value |
|---|---|
| Report ID | INC-2026-001 |
| Date/Time Detected | Lab simulation — see `screenshots/alerts-triage/` for timestamped evidence |
| Analyst | Paramananda Gaigaria |
| Severity | High |
| Status | Closed (lab exercise) |
| MITRE ATT&CK ID | T1110 — Brute Force |

## 1. Executive Summary

A high volume of failed RDP authentication attempts against the local `Administrator` account on `WIN10-VICTIM` (192.168.56.20) was detected by a custom Wazuh correlation rule. The activity originated from `192.168.56.10` (Kali attacker VM in this lab) and was consistent with an automated password-spraying attack using Hydra. The account was not compromised — all attempts failed against the lab's configured password.

## 2. Detection

Wazuh custom rule `100102` fired at **level 10**, triggered by 8+ Windows Security Event ID 4625 ("An account failed to log on") entries from the same source IP within a 120-second window. The alert appeared in the Kibana `wazuh-alerts-*` index and on the Wazuh dashboard's active alerts panel.

## 3. Timeline

| Time (relative) | Event |
|---|---|
| T+0s | Hydra brute-force initiated from Kali against `192.168.56.20:3389` |
| T+4s | First 4625 failed logon events begin arriving at the Wazuh manager |
| T+22s | Rule `100102` threshold (8 failures / 120s) met — alert generated |
| T+25s | Alert visible in Wazuh dashboard and Kibana |
| T+90s | Attack simulation manually stopped |

## 4. Technical Details

Attacker command:

```bash
hydra -l administrator -P /usr/share/wordlists/rockyou.txt rdp://192.168.56.20
```

Representative raw log field values from the matched events:
- `win.system.eventID`: `4625`
- `win.eventdata.targetUserName`: `Administrator`
- `win.eventdata.ipAddress`: `192.168.56.10`
- `win.eventdata.logonType`: `3` (network logon attempt)

## 5. Indicators of Compromise (IOCs)

| Type | Value |
|---|---|
| Source IP | 192.168.56.10 |
| Target Account | Administrator |
| Target Host | WIN10-VICTIM (192.168.56.20) |
| Attack Tool Signature | High-frequency 4625 events, single source, single target account |

## 6. Impact Assessment

No successful authentication occurred; no credentials were compromised in this simulation. In a production environment, this pattern against a privileged local account would carry high risk given RDP's common use as an initial-access and lateral-movement vector.

## 7. Containment & Remediation

Recommended actions (as would apply in production):
- Rate-limit or geofence RDP access; prefer VPN + MFA in front of RDP rather than direct exposure.
- Enforce account lockout policy after N failed attempts.
- Rename or disable the built-in local `Administrator` account.
- Add the offending source IP to a watchlist/blocklist and monitor for repeat activity.

## 8. Lessons Learned / Follow-up

- Rule `100102`'s 120-second window worked well for this attack speed; a slower, low-and-slow brute force (e.g. 1 attempt every 30s) would evade this threshold — noted as a detection gap.
- Follow-up: add a second, longer-window rule (e.g. 5+ failures over 30 minutes) to catch throttled brute-force attempts designed to stay under standard thresholds.
