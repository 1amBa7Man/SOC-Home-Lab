# Setting Up ELK / Kibana Dashboards

The Wazuh all-in-one installer already ships an Elasticsearch (indexer) + Kibana-based dashboard (OpenSearch Dashboards under the hood). This doc covers building your **own** Kibana visualizations on top of that data — the part that actually demonstrates analyst skill.

## 1. Access the dashboard

Browse to `https://192.168.56.30` and log in with the admin credentials generated during install.

## 2. Confirm the index pattern

Wazuh alerts land in the `wazuh-alerts-*` index. Under **Stack Management → Index Patterns**, confirm it exists and `timestamp` is set as the time field.

## 3. Build these dashboards (screenshot each one)

| Dashboard | What it shows | Query/filter |
|---|---|---|
| Authentication Failures | Windows 4625 events over time, by source IP | `data.win.system.eventID: 4625` |
| Process Creation Timeline | Sysmon Event ID 1 process creates | `data.win.eventdata.image: *` |
| Network Scan Detection | Sysmon Event ID 3 network connections spiking from one host | `rule.groups: sysmon_event3` |
| Top MITRE Techniques Fired | Alerts grouped by `rule.mitre.id` | aggregation on `rule.mitre.id` |
| Agent Health | Active vs disconnected agents | `agent.name` |

## 4. Save each as a visualization, then combine into one "SOC Overview" dashboard

This single dashboard is the centerpiece screenshot for `screenshots/elk-kibana/` — it's what you pull up first in an interview when asked "what does your lab look like?"

## 5. (Optional) Standalone ELK for deeper practice

If you want raw Elasticsearch/Logstash/Kibana experience outside Wazuh's bundled stack, install the open-source ELK stack separately and use Filebeat/Winlogbeat to ship Windows + Sysmon logs directly to Logstash for custom parsing (grok/ingest pipelines). This is a strong "extra credit" addition — note it in your README roadmap if you build it.
