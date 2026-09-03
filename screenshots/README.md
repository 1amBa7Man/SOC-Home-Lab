# 📸 Screenshot Evidence

This folder is what turns this repo from "a config on GitHub" into proof you ran the lab. Recruiters and interviewers skim screenshots before they read a line of markdown — fill every subfolder below.

Drag and drop your images into the matching folder (PNG/JPG, descriptive filenames like `01-nmap-scan-detected.png`). Then reference them in the relevant doc/report using:

```markdown
![Wazuh alert for brute force](../screenshots/wazuh-dashboard/01-brute-force-alert.png)
```

## Checklist — aim for at least one image per row

| Folder | What to capture |
|---|---|
| `wazuh-dashboard/` | Wazuh main dashboard, agent list showing your Windows 10 endpoint as active, a fired alert detail view |
| `elk-kibana/` | Kibana Discover view with live Sysmon logs, a custom visualization/dashboard panel |
| `sysmon-events/` | Windows Event Viewer showing a raw Sysmon Event ID 1 (process create) and Event ID 3 (network connect) |
| `osquery/` | Terminal output of an osquery live query (e.g. `SELECT * FROM processes;` or `scheduled_tasks`) |
| `attack-simulations/` | Kali terminal running the actual attack — Nmap scan, Hydra brute force, etc. |
| `alerts-triage/` | The moment the attack shows up as an alert — side-by-side of attacker terminal + Wazuh/Kibana alert firing |

## Why this matters for interviews

When an interviewer asks "show me something you built," you open this folder — not your memory. Six well-chosen screenshots beat a paragraph of description every time.
