# Deploying Sysmon on the Windows 10 Victim

## 1. Download

Get Sysmon from Microsoft Sysinternals (`Sysmon64.exe`).

## 2. Use a maintained community config

Don't hand-roll your filtering rules from scratch — start from the well-known SwiftOnSecurity Sysmon config baseline, then tune it. A starting config is included at [`configs/sysmon/sysmonconfig.xml`](../configs/sysmon/sysmonconfig.xml).

## 3. Install with the config

```powershell
.\Sysmon64.exe -accepteula -i sysmonconfig.xml
```

## 4. Verify it's logging

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5
```

Screenshot Event Viewer showing a live Event ID 1 (process creation) for `screenshots/sysmon-events/`.

## 5. Key event IDs to know cold for interviews

| Event ID | Meaning | Why a SOC analyst cares |
|---|---|---|
| 1 | Process creation | Core telemetry — command line, parent process, hashes |
| 3 | Network connection | Beaconing, port scans, lateral movement |
| 7 | Image loaded (DLL) | DLL sideloading / injection detection |
| 8 | CreateRemoteThread | Process injection |
| 10 | ProcessAccess | Credential dumping (lsass.exe access) |
| 11 | FileCreate | Dropped payloads, ransomware note drops |
| 12/13/14 | Registry events | Persistence via Run keys |
| 22 | DNS query | C2 domain resolution |
