# Deploying osquery on the Windows 10 Victim

osquery complements Sysmon: Sysmon tells you what *just happened* (event-driven), osquery tells you the *current state* of the endpoint (queryable, SQL-based) — persistence, autoruns, installed software, running processes.

## 1. Install

Download and install `osquery-5.23.1.msi` from osquery.io.

## 2. Try it interactively

```powershell
osqueryi
osquery> SELECT pid, name, path FROM processes WHERE name = 'powershell.exe';
osquery> SELECT * FROM scheduled_tasks WHERE enabled = 1;
osquery> SELECT * FROM autoexec;
osquery> SELECT * FROM logged_in_users;
```

Screenshot one of these for `screenshots/osquery/`.

## 3. Schedule the query pack for continuous monitoring

Copy [`configs/osquery/osquery.conf`](../configs/osquery/osquery.conf) to `C:\Program Files\osquery\osquery.conf` and start the daemon service:

```powershell
osqueryd --config_path="C:\Program Files\osquery\osquery.conf" --install
NET START osqueryd
```

## 4. Forward osquery results into the SIEM

Two options:
- **Wazuh's built-in osquery module** — add an `<wodle name="osquery">` block to `ossec.conf` pointing at the osquery results log.
- **Filebeat** with the osquery module, shipping directly to Elasticsearch/Logstash.

Either way, the goal is the same: osquery's point-in-time state checks (e.g. "is there a suspicious scheduled task right now?") become searchable alongside Sysmon's real-time event stream in Kibana.
