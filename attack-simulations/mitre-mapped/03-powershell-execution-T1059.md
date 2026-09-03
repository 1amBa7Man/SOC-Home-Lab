# Scenario 3: Suspicious PowerShell Execution — T1059.001

## Attacker action

After gaining initial access (or simulated locally on the victim for lab purposes), run a Base64-encoded PowerShell download cradle:

```powershell
powershell.exe -nop -w hidden -enc <Base64EncodedPayload>
```

## Telemetry generated

Sysmon **Event ID 1** (Process creation) captures the full command line, including the `-enc` flag and encoded blob, along with parent process (often `cmd.exe`, `winword.exe`, or a scheduled task host).

## Detection

Rule `100103` uses a PCRE2 match against `win.eventdata.commandLine` for common obfuscation/download-cradle indicators (`-enc`, `-encodedcommand`, `-nop -w hidden`, `downloadstring`, `invoke-expression`) and fires a level-12 (high) alert.

## Screenshot checklist

- [ ] `screenshots/sysmon-events/` — raw Event ID 1 in Event Viewer showing the encoded command line
- [ ] `screenshots/elk-kibana/` — the alert visualized in Kibana

## Analyst notes

Encoded PowerShell is one of the highest-signal indicators in Windows telemetry — legitimate admin scripts rarely need obfuscation. Triage steps: decode the Base64 blob (`[System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String($enc))`), identify the parent process, and check whether the resulting network connections (Event ID 3) point to a known-bad destination.
