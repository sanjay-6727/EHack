# Ethical Hacking Lab — Tool Installation Guide
## Windows, Kali Linux, and browser-based tools (excluding VirtualBox and Kali Linux themselves)

This guide covers installing the tools in the current lab toolkit, except VirtualBox and Kali Linux.

**Safety:** Install software from official sources, verify downloads when checksums/signatures are provided, and use scanning/testing tools only on systems you own or are explicitly authorized to assess. Keep Metasploitable 2 isolated on a host-only lab network.

---

# 1. Windows tools

## 1.1 Sysmon — Windows system monitoring

**Purpose:** Records detailed Windows system events for troubleshooting and security investigations.

**Official download:** https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

### Install
1. Download Sysmon from Microsoft's Sysinternals page.
2. Extract the ZIP into a folder, for example `C:\Tools\Sysmon`.
3. Open PowerShell **as Administrator**.
4. Change to the extracted folder:

```powershell
cd C:\Tools\Sysmon
```

5. Install Sysmon:

```powershell
.\Sysmon64.exe -accepteula -i
```

If the executable has a different name, use the actual filename in the folder.

### Verify
```powershell
Get-Service *Sysmon*
```

Open Event Viewer with `eventvwr.msc`, then navigate to **Applications and Services Logs → Microsoft → Windows → Sysmon → Operational**.

### Useful commands
```powershell
.\Sysmon64.exe -c
.\Sysmon64.exe -s
```

- `-c` displays the active configuration.
- `-s` displays the configuration schema.

To uninstall intentionally:

```powershell
.\Sysmon64.exe -u
```

Uninstalling removes the Sysmon service and driver; it does not necessarily delete historical Windows event logs.

---

## 1.2 Nessus — vulnerability scanner

**Purpose:** Scans authorized hosts for potential vulnerabilities, outdated software, and configuration weaknesses.

**Official download:** https://www.tenable.com/downloads/nessus  
**Getting started:** https://docs.tenable.com/nessus/Content/GettingStarted.htm

### Install
1. Open Tenable's official download page.
2. Choose the Nessus edition and platform appropriate for your use. Follow current registration and activation instructions.
3. Download the Windows installer.
4. Run the installer and complete the setup wizard.
5. Open the local web interface, commonly `https://localhost:8834`.
6. Complete setup and activation as prompted.

### Verify
```powershell
Get-Service *nessus*
```

Open the local interface and sign in. If it does not load, check the service status and installation logs.

### First safe scan
1. Go to **Scans → New Scan**.
2. Choose an appropriate template, such as a basic network scan.
3. Set the target to a lab VM you control.
4. Save and launch the scan.
5. Review findings, evidence, severity, and remediation.

Do not scan public or third-party systems without explicit authorization.

---

## 1.3 FOCA — document metadata analysis

**Purpose:** Examines document metadata and, depending on version, supports workflows for discovering publicly available documents.

**Project page:** https://github.com/ElevenPaths/FOCA

### Install
1. Read the project's current README and releases.
2. Download a release or follow its build instructions.
3. Check the release requirements, including the supported Windows version and required .NET runtime/framework.
4. Extract the package if supplied as an archive, or run the installer if provided.
5. Launch the application.

FOCA is an older project; release availability, dependencies, and compatibility may differ. Avoid unofficial repackaged executables.

### Verify
Create a project and analyze a document you created yourself.

---

## 1.4 Maltego — OSINT relationship graphing

**Purpose:** Maps relationships among entities such as domains, IP addresses, and organizations.

**Official website/downloads:** https://www.maltego.com/  
**Documentation:** https://docs.maltego.com/

### Install
1. Download Maltego Graph for Windows from the official website.
2. Run the installer.
3. Launch the application.
4. Sign in or create an account if prompted, and choose the edition available to you.
5. Review the available transforms and data-provider requirements.

### Verify
Create a new graph, add a Domain entity, and inspect the available transforms. Some transforms require provider accounts, API keys, credits, or specific plan permissions.

---

# 2. Kali Linux command-line tools

Use a terminal in Kali. Run `sudo apt update` before installing packages. Package names and availability can change between Kali releases.

## 2.1 Nmap — host, port, and service discovery

**Purpose:** Discovers hosts and examines ports and services on authorized targets.

**Official site:** https://nmap.org/  
**Kali package page:** https://www.kali.org/tools/nmap/

### Install and verify
```bash
sudo apt update
sudo apt install nmap
nmap --version
nmap -h
```

### Safe lab examples
```bash
nmap -sn LAB_SUBNET/CIDR
nmap -sV TARGET_IP
```

Replace placeholders with the subnet and target IP from your isolated lab. `-sn` performs host discovery without a port scan; `-sV` attempts service/version detection.

---

## 2.2 WHOIS — registration and IP allocation lookup

**Purpose:** Retrieves available domain registration or IP allocation records.

### Install and verify
```bash
sudo apt update
sudo apt install whois
whois --version
```

### Example
```bash
whois example.com
```

WHOIS data may be incomplete or redacted. A registrar is not necessarily the hosting provider.

---

## 2.3 nslookup — DNS queries

`nslookup` is commonly included with DNS utilities. If it is missing on Kali, try:

```bash
sudo apt update
sudo apt install dnsutils
```

If your release does not provide that package name, search for the provider package:

```bash
apt-cache search nslookup
```

### Verify and try a query
```bash
nslookup -help
nslookup example.com
nslookup -type=MX example.com
nslookup -type=NS example.com
```

Some versions display usage information differently.

---

## 2.4 Netdiscover — local ARP discovery

**Purpose:** Discovers hosts on a local IPv4 network, commonly using ARP.

### Install and verify
```bash
sudo apt update
sudo apt install netdiscover
netdiscover -h
```

### Identify your interface and subnet
```bash
ip -br addr
ip route
```

Then use the interface/subnet actually assigned to your host-only lab:

```bash
sudo netdiscover -i INTERFACE_NAME
```

Or:

```bash
sudo netdiscover -r LAB_SUBNET/CIDR
```

Netdiscover is mainly useful on the local network segment; ARP discovery does not ordinarily traverse routers.

---

## 2.5 theHarvester — public-source reconnaissance

**Purpose:** Collects publicly available domain, host, and email information from supported sources.

**Kali package page:** https://www.kali.org/tools/theharvester/

### Install and verify
```bash
sudo apt update
sudo apt install theharvester
theHarvester -h
```

Check the help output for sources supported by your installed version. A source-specific example, if supported:

```bash
theHarvester -d example.com -b crtsh
```

Some sources require API keys or have rate limits. Use only an authorized domain and respect provider terms.

---

## 2.6 Sublist3r — subdomain enumeration

**Purpose:** Finds candidate subdomains using supported public sources and, in some versions, brute-force techniques.

Sublist3r is an older project, and individual source integrations may no longer work reliably.

### Check package availability
```bash
sudo apt update
apt-cache policy sublist3r
```

If a candidate package is available:

```bash
sudo apt install sublist3r
sublist3r -h
```

### If using a downloaded source copy
Use the project's trusted repository and read its README for the correct dependencies and installation procedure. Avoid random ZIP files or unofficial binaries.

A source copy may be checked with:

```bash
python3 sublist3r.py -h
```

Run from the directory containing the script. Follow that version's requirements file or README for Python dependencies.

### Troubleshooting
A source may fail because a search engine changed its page, access is restricted, or the provider requires authentication. A scraper error does not prove that no subdomains exist.

---

## 2.7 Metasploit Framework — security testing framework

**Purpose:** Provides modules for authorized security assessment and controlled validation in a lab.

**Official documentation:** https://docs.metasploit.com/  
**Kali package page:** https://www.kali.org/tools/metasploit-framework/

### Install and verify
```bash
sudo apt update
sudo apt install metasploit-framework
msfconsole --version
```

Launch the console:

```bash
msfconsole
```

Inside the console, basic informational commands include:

```text
help
version
exit
```

### Database
```bash
sudo msfdb status
```

If needed and supported:

```bash
sudo msfdb init
```

Database setup varies by version. Follow the command output and current documentation.

**Safety:** Use exploit modules only against deliberately vulnerable systems you own or have explicit permission to test.

---

## 2.8 Armitage — Metasploit graphical interface

**Purpose:** An older GUI for interacting with Metasploit and organizing discovered hosts.

**Kali package page:** https://www.kali.org/tools/armitage/

### Check availability
```bash

sudo apt update
sudo apt install postgresql
sudo systemctl enable --now postgresql
sudo systemctl status postgresql

sudo apt update
apt-cache policy armitage
```

If a candidate package is available:

```bash
sudo apt install armitage
```

Metasploit is a separate dependency and may be installed with:

```bash
sudo apt install metasploit-framework
```

### Verify and launch
```bash
which armitage
armitage
```

Armitage may not be packaged or fully compatible with every modern Kali release. If unavailable, do not install a random repackaged copy. Use the official package information and Metasploit console as a fallback.

---

## 2.9 Wireshark — packet capture and analysis

**Purpose:** Captures and decodes network packets for protocol troubleshooting and security analysis.

**Official downloads:** https://www.wireshark.org/download.html  
**Documentation:** https://www.wireshark.org/docs/

### Install and verify
```bash
sudo apt update
sudo apt install wireshark
wireshark --version
tshark --version
tshark -D
```

- `wireshark` launches the GUI.
- `tshark` is the terminal-oriented capture/analysis utility.
- `tshark -D` lists capture interfaces.

If prompted about allowing non-root users to capture packets, choose according to your lab's security policy. Avoid running general desktop applications as root merely to capture traffic.

### First exercise
1. Open Wireshark.
2. Select the interface carrying your lab traffic.
3. Start capture and perform a DNS lookup.
4. Stop capture and apply the display filter `dns`.
5. Inspect the query and response.

Capture only traffic you are authorized to monitor.

---

# 3. Browser-based tools — no local installation required

## 3.1 Shodan

**Purpose:** Searches collected observations of internet-facing services and devices.

**Website:** https://www.shodan.io/

1. Open the site.
2. Create/sign in to an account if needed.
3. Search an authorized hostname or IP.
4. Inspect observed ports, banners, and timestamps.
5. Verify findings against your own inventory.

Examples of common search filters:

```text
hostname:example.com
port:443
country:IN
```

Shodan data may be stale or incomplete. A result is not permission to access or test the device.

## 3.2 Robtex

**Purpose:** Browser-based DNS and IP relationship lookup.

**Website:** https://www.robtex.com/

1. Search an authorized domain or IP.
2. Inspect DNS records and relationships.
3. Cross-check key results with `nslookup` or another reliable source.

No local installer is needed.

## 3.3 Google Dorking

**Purpose:** Uses advanced search operators to narrow public search results.

**Website:** https://www.google.com/

Examples:

```text
site:example.com
site:example.com filetype:pdf
site:example.com intitle:documentation
site:example.com inurl:docs
```

Use these for authorized public-information research. Do not search for or access third-party secrets or confidential material.

---

# 4. Target VM — Metasploitable 2

Metasploitable 2 is an intentionally vulnerable target VM, not a tool installed inside Kali.

**Project/download page:** https://sourceforge.net/projects/metasploitable/

### High-level setup
1. Download the official archive.
2. Extract it on Windows.
3. Create/import a VirtualBox VM using the supplied virtual disk.
4. Attach the target to the same **host-only** network as Kali's lab adapter.
5. Do not use bridged networking or expose the target through port forwarding.
6. Boot the VM and identify its IP from its console.
7. Test connectivity from Kali and scan only that lab IP.

The project's documented default lab credentials may be `msfadmin` / `msfadmin`. These are for the intentionally vulnerable training VM only.

---

# 5. Installation verification checklist

## Windows PowerShell

```powershell
Get-Service *Sysmon*
Get-Service *nessus*
```

Service names can vary. Open the relevant GUIs and verify:
- Sysmon events appear in Event Viewer after a test event.
- Nessus local web interface loads.
- FOCA opens and can analyze a sample document.
- Maltego opens and displays available transforms.

## Kali terminal

```bash
nmap --version
whois --version
nslookup -help
netdiscover -h
theHarvester -h
msfconsole --version
wireshark --version
tshark --version
```

Check Armitage and Sublist3r if installed:

```bash
which armitage
sublist3r -h
```

A missing command can mean the package is unavailable, installed under another name, or not on PATH.

---

# 6. Recommended installation order

1. Set up and isolate Metasploitable 2.
2. Install/verify Nmap, WHOIS, DNS utilities, and Netdiscover.
3. Install/verify Wireshark.
4. Install/verify Metasploit.
5. Attempt Armitage only if your Kali release provides a compatible package.
6. Install theHarvester and check Sublist3r availability.
7. Install Sysmon on Windows.
8. Install Nessus and complete activation.
9. Install FOCA and Maltego if needed for coursework.
10. Bookmark Shodan, Robtex, and Google; no local installation is needed for these websites.

---

# 7. Official source index

- Sysmon: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
- Nessus: https://www.tenable.com/downloads/nessus
- FOCA: https://github.com/ElevenPaths/FOCA
- Maltego: https://www.maltego.com/
- Nmap: https://nmap.org/
- Kali tools: https://www.kali.org/tools/
- Metasploit: https://docs.metasploit.com/
- Armitage: https://www.kali.org/tools/armitage/
- Wireshark: https://www.wireshark.org/
- Shodan: https://www.shodan.io/
- Robtex: https://www.robtex.com/
- Metasploitable 2: https://sourceforge.net/projects/metasploitable/

**Remember:** Use official sources, verify versions, keep the vulnerable target isolated, and only test systems you are authorized to assess.
