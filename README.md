# Zabbix Windows Event Log Template

Zabbix template for monitoring Windows events log through active agent checks.

Import [template_windows_log.yaml](template_windows_log.yaml), exported in **Zabbix 7.0** format. The template is named `Template Windows Event Log`, belongs to the `Templates/Operating systems` group.

## Monitored events

The template includes three `ZABBIX_ACTIVE` items with the `LOG` value type and three triggers:

| Item                      | Windows log | Source filter | Event ID filter | Windows level     | Trigger severity |
| ------------------------- | ----------- | ------------- | --------------- | ----------------- | ---------------- |
| Commvault - Error event   | Application | ContentStore  | `256`           | Error or Critical | HIGH             |
| Commvault - Warning event | Application | ContentStore  | `256`           | Warning           | WARNING          |
| BlueScreen                | System      | Any           | `6008`          | Error or Critical | HIGH             |

`BlueScreen`: Windows event 6008 indicates an unexpected shutdown and does not, by itself, confirm that a BSOD occurred. See the [Microsoft guide to unexpected shutdowns](https://learn.microsoft.com/en-us/troubleshoot/windows-server/performance/troubleshoot-unexpected-reboots-system-event-logs).

`Commvault log`: These checks monitor Commvault alert notifications written to the Windows Application log with source ContentStore and Event ID 256. `Commvault - Error event`: captures events with Windows level Error or Critical and raises a Zabbix problem with HIGH severity. `Commvault - Warning event`: captures events with Windows level Warning and raises a Zabbix problem with WARNING severity.
Event ID 256 alone does not identify a specific failure or prove that a backup failed. Read the event message to determine the reported condition, then investigate the corresponding alert or job in Commvault. Both trigger names include {ITEM.LASTVALUE} to display the event message. See the [Commvault guide](https://documentation.commvault.com/v11/commcell-console/setting_up_windows_event_viewer_alert_notifications.html)

## Prerequisites

- A Zabbix installation that can import the 7.0 export format. Compatibility with other versions must be verified in the target environment.
- Zabbix agent on Windows, configured for active checks and permitted to read the `Application` and `System` logs.
- Connectivity from the agent to the configured Zabbix server or proxy, normally over TCP port 10051.
- For the Commvault checks, events matching the `ContentStore` source in the `Application` log.

No external scripts, UserParameter settings, or template-specific user macros are required.

## Configuration and import

1. Set the active check parameters in the Windows agent configuration file. Adapt this example to your environment:

   ```ini
   ServerActive=zabbix.example.local
   Hostname=WINDOWS-SERVER-01
   ```

   `Hostname` must match the technical host name configured in Zabbix. If the host is monitored through a proxy, specify that proxy in `ServerActive`. Restart the agent service after making changes. For details, see the [Zabbix guide to monitoring Event Logs with active checks].

2. In the Zabbix frontend, open **Data collection → Templates → Import** and select `template_windows_log.yaml`.
3. Review the import preview and import the template.
4. Under **Data collection → Hosts**, create or open the Windows host, link `Template Windows Event Log`, and save. Make sure the host and its items are enabled.
5. Check new events under **Monitoring → Latest data** and alerts under **Monitoring → Problems**.

## Customization

To add more events, duplicate an item and its trigger, then adjust the log, source, Event ID, levels, and severity. If you change an item key, update its reference in the trigger expression. When editing the YAML file directly, assign unique UUIDs to new objects; alternatively, edit the template in the frontend and export it again.

On hosts that do not use Commvault, you can disable the two dedicated items.
