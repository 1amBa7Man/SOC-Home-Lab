# Writing Custom Wazuh Detection Rules

This is the part of the lab that separates "I installed a SIEM" from "I can do detection engineering" — the skill that actually gets SOC Analyst offers.

## The workflow used for every rule in this repo

1. **Run the attack** from `attack-simulations/` against the Windows 10 victim.
2. **Find the raw log** it generated — either a Sysmon event or a Windows Security event — in `/var/ossec/logs/archives/archives.json` on the manager (archives must be enabled in `ossec.conf`).
3. **Write a decoder/rule** that matches on the specific fields of that log (event ID, command-line substring, image path).
4. **Reload the manager** and re-run the attack to confirm the alert fires.
5. **Document it**: attacker command → raw log excerpt → rule XML → screenshot of the fired alert → MITRE ATT&CK ID.

## Example: detecting encoded PowerShell (T1059.001)

Attacker runs:

```powershell
powershell.exe -enc SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAApAC4A...
```

This generates a Sysmon Event ID 1 with `-enc` in the command line. The matching rule ([`configs/wazuh-rules/local_rules.xml`](../configs/wazuh-rules/local_rules.xml)) looks for that substring in `win.eventdata.commandLine` and fires at a `level="12"` (high severity), tagged with MITRE technique `T1059.001`.

## Rule severity levels used in this lab

| Level | Meaning | Example |
|---|---|---|
| 3–5 | Informational / low | Successful logon, normal process start |
| 7–9 | Suspicious | Port scan detected, unusual parent-child process pair |
| 10–12 | High | Encoded PowerShell, lsass access, brute-force threshold hit |
| 13+ | Critical | Confirmed credential dump tool signature, ransomware IOC |

## Where to go next

Once you're comfortable writing rules directly in Wazuh's XML syntax, the natural next step is learning **Sigma rules** (vendor-agnostic detection-as-code) and converting them to Wazuh format with `sigma2wazuh` or `sigmac`. This is listed on the roadmap in the main README.
