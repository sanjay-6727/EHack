# PostgreSQL for Metasploit and Armitage

## 1. Purpose
PostgreSQL is a relational database server. Metasploit can use it to persist assessment workspace data such as hosts, services, and notes. Armitage may rely on the Metasploit setup/database features.

**Memory hook:** PostgreSQL = **the notebook**; Metasploit = **the engine**; Armitage = **the dashboard**.

## 2. Inputs
- PostgreSQL package/service.
- Metasploit Framework and its supported database configuration.
- Local service permissions/configuration.
- No need to invent a separate database/user if the supported Kali `msfdb` setup handles it.

## 3. Install and start on Kali
```bash
sudo apt update
sudo apt install postgresql
sudo systemctl enable --now postgresql
sudo systemctl status postgresql
```
If Metasploit is not installed:
```bash
sudo apt install metasploit-framework
```
Initialize/check the Metasploit database using the helper available in your version:
```bash
sudo msfdb init
sudo msfdb status
```
Then:
```bash
msfconsole
```
At the `msf>` prompt:
```text
db_status
```

## 4. Expected output and interpretation
- `systemctl status`: service state and recent logs. `active (running)` is common; packaging/setup can sometimes show other states, so inspect the full status and logs.
- `msfdb status`: reports database/helper configuration status for the installed version.
- `db_status` in Metasploit: should indicate whether the framework is connected to its database.
A running PostgreSQL service alone does not guarantee Metasploit is connected.

## 5. Troubleshooting
- `msfdb: command not found`: check whether Metasploit is installed and how the installed version manages its database.
- Database connection error: check `sudo systemctl status postgresql`, then `sudo msfdb status` and `db_status`.
- Permission/authentication error: use the supported setup path rather than randomly creating users or changing authentication rules.
- Armitage RPC/database issue: fix the backend first; then relaunch Armitage and follow its version-specific connection prompts.

## 6. Viva
**Q: What is PostgreSQL?** An open-source relational database management system.
**Q: Why does Metasploit use it?** To store and query workspace/assessment data persistently.
**Q: Does `active` prove Metasploit is connected?** No; verify with `db_status`.
**Q: Why not expose the database port?** It is unnecessary for a local lab and increases attack surface.
