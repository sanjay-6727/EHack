# Armitage — GUI for Metasploit

## 1. Purpose
Armitage is a graphical interface for interacting with Metasploit. It can visualize hosts and services and help organize an authorized lab assessment. It is an older project, so availability and compatibility vary across Kali versions.

**Memory hook:** Armitage = **the control panel**; Metasploit = **the engine**.

## 2. Inputs / prerequisites
- Kali Linux.
- Metasploit Framework (`msfconsole`).
- PostgreSQL and a working Metasploit database, if using database features.
- The lab target IP or imported scan results.
- Authorization to assess the target.

## 3. Check installation and database
```bash
msfconsole --version
sudo systemctl status postgresql
sudo msfdb status
```
If your Kali installation supports the helper:
```bash
sudo msfdb init
```
Then launch:
```bash
armitage
```
Some versions may require a different launch procedure or may no longer package Armitage. Check `apt-cache policy armitage` before assuming it is available.

## 4. GUI workflow
1. Start the database and Metasploit service/setup as appropriate for the installed version.
2. Launch Armitage.
3. Connect to the local Metasploit RPC/service when prompted; use the settings expected by your installed version.
4. Add or scan only the authorized lab host.
5. Review the host and service list.
6. Use the GUI to inspect results. Do not select exploit actions merely because a menu suggests them; verify the target and module first.

## 5. Output and interpretation
- A host icon means a host has been added/discovered in the workspace.
- Service entries represent detected or imported services, not proof that they are exploitable.
- A connection/database error means the GUI cannot reach a required backend; it does not mean the target is the problem.

**Viva wording:** “Armitage provides a graphical workflow over Metasploit. It can display discovered hosts and services, while actual module execution is handled by the underlying framework.”

## 6. Troubleshooting
- `command not found`: package may be unavailable in this Kali release; inspect `apt-cache policy armitage`.
- Database not connected: inspect PostgreSQL and `msfdb status`; then check `db_status` in `msfconsole`.
- RPC connection failure: check the endpoint/port and credentials expected by the installed version; avoid exposing RPC outside the lab.
- GUI opens but no hosts appear: perform/import an authorized discovery step and refresh the workspace.

## 7. Viva
**Q: Is Armitage a separate exploit engine?** No, it is a GUI for Metasploit.
**Q: Why PostgreSQL?** To persist workspace data such as hosts, services, and findings.
**Q: Does a listed service prove a vulnerability?** No; it is an observation that needs validation.
**Q: Is Armitage modern and guaranteed to work?** No; it is older and version compatibility varies.
