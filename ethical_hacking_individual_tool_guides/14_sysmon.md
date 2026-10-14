# Sysmon — Windows System Activity Monitoring

## 1. Purpose
Sysmon is a Windows system service and driver that logs detailed system activity to the Windows Event Log, useful for detection and investigation.

**Memory hook:** Sysmon = **a CCTV logbook for Windows activity**.

## 2. Inputs
- Windows machine and administrator rights for installation/configuration.
- Sysmon executable from Microsoft's Sysinternals distribution.
- Optional XML configuration controlling which event types are collected.

## 3. Install / verify / uninstall
Run PowerShell as Administrator in the folder containing the executable:
```powershell
.\Sysmon64.exe -accepteula -i
Get-Service *Sysmon*
```
To uninstall, use the executable's documented uninstall option:
```powershell
.\Sysmon64.exe -u
```
The exact executable name may differ by package/architecture. If you already ran `-u` and saw the service/driver removed, that indicates uninstall succeeded.

## 4. View output
1. Press Win+R → type `eventvwr.msc`.
2. Open **Applications and Services Logs → Microsoft → Windows → Sysmon → Operational**.
3. Find recent events and open **Details** or **General**.
4. A common event is **Event ID 1 — Process creation**. Generate a benign event by opening Notepad, then inspect the matching timestamp.

## 5. Output interpretation
Event ID 1 commonly includes process image/path, command line (depending on configuration), parent process, user, and process identifiers. Interpret the timestamp and process lineage together; one event alone does not prove malicious activity.

## 6. Common problems
- No Sysmon log: service may not be installed/running, channel may be wrong, or config may filter events.
- No expected event: configuration rules can exclude it.
- Uninstall leaves old events: removing the service does not necessarily delete historical log records.

## 7. Viva
**Q: What does Sysmon do?** Produces detailed event telemetry for monitoring and investigation.
**Q: Sysmon vs Event Viewer?** Sysmon generates telemetry; Event Viewer displays event logs.
**Q: What is process creation?** An event describing a process being started.
**Q: Does Sysmon block malware?** No, it primarily logs; prevention requires other controls.
