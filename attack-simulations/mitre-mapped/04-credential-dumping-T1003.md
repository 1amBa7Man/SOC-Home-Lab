# Scenario 4: OS Credential Dumping — T1003

## Attacker action

Simulate a Mimikatz-style credential dump against LSASS (use a lab-safe procdump approach if you prefer not to run real Mimikatz):

```powershell
procdump64.exe -accepteula -ma lsass.exe lsass_dump.dmp
```

or, with actual offensive tooling delivered from the Kali box, `sekurlsa::logonpasswords` via Mimikatz.

## Telemetry generated

Sysmon **Event ID 10** (ProcessAccess) logs any process opening a handle to `lsass.exe` with a memory-read access mask — this is true regardless of which specific dumping tool is used.

## Detection

Rule `100104` matches on `win.eventdata.targetImage` containing `lsass.exe` in a ProcessAccess event and fires a **level-13 critical** alert — the highest severity rule in this lab, appropriately, since successful LSASS access is one of the most damaging outcomes a SOC analyst can face.

## Screenshot checklist

- [ ] `screenshots/attack-simulations/` — the dumping tool executing
- [ ] `screenshots/wazuh-dashboard/` — the critical alert, with the source process and access mask visible

## Analyst notes

This is a "drop everything" alert. Correct triage/response: isolate the host immediately, identify the accessing process and its parent, check for local admin/SeDebugPrivilege abuse, and assume the account's credentials (and any cached domain credentials) are compromised — force a password reset and check for lateral movement using those credentials afterward.
