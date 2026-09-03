# Setting Up Wazuh Manager + Windows Agent

## 1. Install the Wazuh Manager (on SOC-Server, Ubuntu 22.04/24.04)

```bash
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

This installs the Wazuh indexer, manager, and dashboard in a single-node deployment. Note the admin password printed at the end.

Verify the manager is running:

```bash
sudo systemctl status wazuh-manager
```

## 2. Install the Wazuh Agent on the Windows 10 victim

Download the latest Wazuh agent MSI from the Wazuh downloads page, then install via PowerShell (as Administrator):

```powershell
msiexec.exe /i wazuh-agent.msi /q WAZUH_MANAGER='192.168.56.30' WAZUH_AGENT_NAME='WIN10-VICTIM'
NET START WazuhSvc
```

## 3. Confirm the agent checks in

On the manager:

```bash
sudo /var/ossec/bin/agent_control -l
```

You should see `WIN10-VICTIM` listed as **Active**. Screenshot this for `screenshots/wazuh-dashboard/`.

## 4. Point Wazuh at Sysmon logs

Edit the agent's `ossec.conf` (`C:\Program Files (x86)\ossec-agent\ossec.conf`) and add a `<localfile>` block for the Sysmon Operational channel:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Restart the agent service after saving.

## 5. Load custom detection rules

Copy the rule files from [`configs/wazuh-rules/`](../configs/wazuh-rules/) into `/var/ossec/etc/rules/` on the manager, then restart:

```bash
sudo systemctl restart wazuh-manager
```

Test a rule fires by triggering the matching attack from `attack-simulations/`, then check `/var/ossec/logs/alerts/alerts.json`.
